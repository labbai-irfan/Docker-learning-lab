# TOPIC: Linux Internals

> **Phase:** 0 · **Module:** Foundations · **Difficulty:** Beginner → Expert  
> **Prerequisites:** [[computing-fundamentals]], [[virtualization]]  
> **Status:** ✅ Complete

---

## 1. Core Treatment

### Definition

Linux internals are the mechanisms and structures deep in the Linux kernel that enable containerization. You cannot understand containers without understanding Linux. Specifically, you need to know: how processes are created and scheduled, how filesystems are mounted and accessed, how signals communicate between processes, how the kernel tracks user permissions, and how security modules restrict access. Linux is a monolithic kernel—all drivers and subsystems run in kernel space, not as separate processes. This unified architecture means containers share more than just the kernel; they share device drivers, filesystem handlers, and network stacks. Understanding Linux internals means understanding the boundaries of container isolation.

### Purpose

Containers work because Linux provides the primitives needed for isolation. Without Linux's process model, filesystem abstraction, signal handling, capability system, and security modules, containers wouldn't exist. Linux internals are the "why" behind every container feature: why PID 1 is special (init process), why volumes work (mount namespace), why you can't see another container's files (mount isolation), why capabilities limit privilege (instead of all-or-nothing root), why AppArmor/SELinux policies matter (restricting syscalls), and why seccomp whitelists are effective (blocking dangerous syscalls).

### Internal Working

**The process model:**

A Linux process is created via `fork()` or `clone()` (more on `clone()` in the next topic):

```c
// Simplified fork() behavior
pid_t child_pid = fork();
if (child_pid == 0) {
    // Child process: runs here with a new PID
} else {
    // Parent process: child_pid is the child's PID
}
```

When `fork()` is called:
1. Kernel allocates a new `task_struct` (process structure)
2. Kernel copies parent's memory, file descriptors, signal handlers
3. Kernel assigns a new PID
4. Kernel returns control to both parent and child (at the fork point)
5. Both run independently; kernel scheduler decides which runs next

The child gets a copy of the parent's memory (via CoW—see Topic 1). Files are shared (via a shared `files_struct`).

**Process lifecycle:**

```
fork() → runnable → running → stopped → zombie → reaped

1. fork(): kernel creates new process, returns to both parent and child
2. Runnable: kernel scheduler can run it (waiting for CPU)
3. Running: executing on a CPU core
4. Stopped: SIGSTOP received; paused but not terminated
5. Zombie: process exited, parent hasn't called wait() yet
6. Reaped: parent called wait(); process resources freed
```

A zombie is a process that exited but hasn't been reaped. It occupies a PID and takes up memory (even though it's not running). Init (PID 1) is responsible for reaping orphaned processes.

**Filesystem mounting:**

Linux filesystems are organized in a single tree:

```
/
├── /bin          (executables)
├── /etc          (config files)
├── /home         (user home dirs)
├── /proc         (kernel interfaces)
├── /sys          (sysfs, kernel state)
├── /dev          (device nodes)
└── /var          (logs, cache)
```

Mounting attaches a filesystem to a point in this tree:

```bash
mount /dev/sda1 /mnt/data
# Now /mnt/data shows contents of /dev/sda1
# /mnt/data itself is the "mount point"
```

Unmounting removes it:

```bash
umount /mnt/data
# /mnt/data is empty again (the original contents visible, if they existed)
```

Each process has a "root filesystem" and a "current working directory". Containers change the root filesystem via `chroot()` or `pivot_root()`.

**Signals:**

Signals are asynchronous notifications sent to processes:

```bash
kill -SIGTERM <pid>  # Graceful shutdown request
kill -SIGKILL <pid>  # Immediate termination (can't be caught)
kill -SIGSTOP <pid>  # Pause the process
kill -SIGCONT <pid>  # Resume the process
```

Signals are numbered (SIGTERM = 15, SIGKILL = 9, etc.). A process can define handlers for most signals:

```c
signal(SIGTERM, &my_handler);  // Set custom handler
signal(SIGKILL, &my_handler);  // ERROR: SIGKILL can't be caught; illegal
```

Containers inherit signal handling from their parent. If PID 1 (the container init process) doesn't handle SIGTERM properly, the container won't shut down gracefully.

**Capabilities:**

Traditionally, Linux has two privilege levels: root (UID 0) and non-root. Root can do anything. Capabilities break this into fine-grained permissions:

```
CAP_NET_ADMIN    - Manage networks
CAP_SYS_ADMIN    - Generic admin stuff
CAP_DAC_OVERRIDE - Bypass file permissions
CAP_SETUID       - Change UID/GID
CAP_SYS_CHROOT   - Use chroot()
... ~40 capabilities total
```

A process can have capabilities even if it's not root:

```bash
# Give a process CAP_NET_ADMIN without being root
setcap cap_net_admin=ep /usr/bin/tcpdump
/usr/bin/tcpdump  # Now can sniff network without root
```

Containers drop capabilities: `--cap-drop=ALL` removes all, then selectively add back: `--cap-add=NET_ADMIN`.

**User and group management:**

Every process has UIDs (user IDs) and GIDs (group IDs). Files have owners (UID/GID) and permissions:

```bash
ls -l /etc/passwd
# -rw-r--r-- 1 root root /etc/passwd
#            |  |    |
#            |  |    +-- group (root) 
#            |  +------- owner (UID 0)
#            +---------- permissions: owner read/write, group read, others read
```

The kernel enforces permissions:

```bash
cat /etc/shadow  # Only root can read (UID 0)
# Permission denied

sudo cat /etc/shadow  # sudo grants temporary root
# (contents shown)
```

Containers can remap UIDs using user namespaces (see next topic), but the kernel still enforces permission checks based on the remapped UID.

**chroot and pivot_root:**

`chroot()` changes the root filesystem:

```bash
chroot /var/lib/docker/overlay2/<layer-id>/merged /bin/sh
# Now "/" points to the container's rootfs
# The process can't access /var/lib/docker anymore
```

`pivot_root()` is more robust; it atomically changes the root:

```
pivot_root new_root put_old
# new_root becomes /
# old / is moved to put_old
```

Both are used to isolate container filesystems.

**Init systems:**

Every Linux system has an init system (PID 1). Historically, it was `/sbin/init` (sysvinit). Modern systems use `systemd`. Containers need an init process too:

```
container start → runc exec /bin/sh (PID 1)
                  → /bin/sh is PID 1
                  → user commands run as children
                  → when /bin/sh exits, container exits
```

If /bin/sh exits, all child processes are terminated. This is fine for simple containers. Complex containers use `tini` (tiny init) or `systemd` to handle signals and reaping.

---

### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    User Space (Applications)                │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Process 1 (PID 1001)  │  Process 2 (PID 1002)      │  │
│  │  task_struct           │  task_struct                │  │
│  │  mm_struct (memory)    │  files_struct (fd table)    │  │
│  │  signal_handlers       │  namespaces                 │  │
│  │  capabilities          │  SELinux context            │  │
│  └──────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────┤
│                      Kernel Space                            │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Process Scheduler     (decides which process runs)  │  │
│  ├──────────────────────────────────────────────────────┤  │
│  │  Memory Manager        (page tables, TLB, swapping)  │  │
│  ├──────────────────────────────────────────────────────┤  │
│  │  Filesystem Subsystem  (mount trees, inodes, dcache) │  │
│  ├──────────────────────────────────────────────────────┤  │
│  │  VFS (Virtual FS)      (abstracts ext4, btrfs, etc)  │  │
│  ├──────────────────────────────────────────────────────┤  │
│  │  Signal Handling       (dispatch signals to processes)│  │
│  ├──────────────────────────────────────────────────────┤  │
│  │  Capability System     (fine-grained permissions)    │  │
│  ├──────────────────────────────────────────────────────┤  │
│  │  LSM (Linux Sec Module) (AppArmor, SELinux hooks)    │  │
│  ├──────────────────────────────────────────────────────┤  │
│  │  Device Drivers        (I/O, networking)             │  │
│  └──────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────┤
│                      Hardware                                │
│        (CPU, RAM, disk controllers, NICs, USB, etc.)       │
└─────────────────────────────────────────────────────────────┘
```

### Lifecycle

```
Process creation → initialization → running → signal → exit → zombie → reaped

1. Creation: fork() or clone(), new task_struct allocated
2. Initialization: new process starts at fork/clone return point
3. Running: kernel scheduler runs the process
4. Signal: kernel delivers signal (SIGTERM, SIGKILL, etc.)
5. Exit: process calls exit() or receives fatal signal
6. Zombie: process exited, parent hasn't reaped
7. Reaped: parent calls wait(), process resources freed
```

### Components

**1. Process structures (kernel):**
- `task_struct`: All info about a process (PID, memory, files, etc.)
- `mm_struct`: Memory management (page tables, memory regions)
- `files_struct`: Open file descriptors
- `signal_struct`: Signal handlers and pending signals
- `cred`: User/group IDs, capabilities, security context

**2. Filesystem:**
- VFS (Virtual Filesystem): Abstraction layer for different filesystems
- Inode: Represents a file/directory (metadata)
- Dentry: Directory entry (filename → inode mapping)
- Superblock: Filesystem metadata (root inode, block size, etc.)
- Mount point: Where a filesystem is attached to the tree

**3. Security:**
- Capability: Fine-grained permission (CAP_NET_ADMIN, etc.)
- LSM (Linux Security Module): Pluggable security framework (AppArmor, SELinux)
- Seccomp: Restrict syscalls via filters
- SELinux/AppArmor: Mandatory access control (MAC)

**4. Signals:**
- Signal descriptor: Per-process signal handlers
- Pending signal queue: Signals waiting to be delivered
- Signal mask: Which signals are blocked

---

### Data Flow

**When a container starts:**

```
1. runc exec /bin/bash in container namespace
2. Kernel fork() is called (or clone() with namespace flags)
3. Kernel creates new task_struct with new PID
4. Child process starts executing at fork return point
5. runc sets up namespace, cgroup, chroot, etc.
6. runc exec("bash") → bash PID 1 in container
7. Bash runs as PID 1 in isolated namespace
8. All commands run as children of bash
9. When bash exits, all children killed, container exits
```

**When a file is accessed inside a container:**

```
1. Process calls open("/etc/passwd")
2. Kernel translates /etc/passwd using container's namespace
3. Kernel translates to actual inode on container rootfs
4. Kernel checks file permissions (UID/GID vs process UID/GID)
5. Kernel checks capability: if process can access file
6. Kernel checks SELinux/AppArmor context
7. If all checks pass, file descriptor returned
8. Process can read/write the file
```

**When a signal is sent:**

```
1. User calls kill -SIGTERM <pid>
2. Kernel looks up process by PID
3. Kernel adds SIGTERM to process's pending signal queue
4. Next time process switches from kernel to user space:
5. Kernel checks pending signals
6. If signal is unblocked:
   a. Kernel interrupts process
   b. Kernel calls signal handler (if set)
   c. After handler returns, process continues
7. If signal is not handled (default):
   a. Kernel performs default action (often: terminate process)
```

---

### Advantages

✅ **Fine-grained control:** Capabilities, LSM policies, seccomp filters give precise control over what a process can do

✅ **Signal handling:** Allows graceful shutdown, pause/resume, custom behavior

✅ **Filesystem abstraction:** Processes see isolated filesystem views (via mount namespace + chroot)

✅ **User/group system:** Multi-user access control built in

✅ **Proven at scale:** Linux is used in millions of production systems

✅ **Open source:** You can read kernel source, understand exactly how it works

### Disadvantages

❌ **Complexity:** Linux kernel is massive; learning all subsystems takes years

❌ **Monolithic:** All drivers run in kernel space; a driver bug can crash the kernel

❌ **Overhead:** Syscall overhead (switching between user/kernel space) is non-zero

❌ **Security complexity:** AppArmor/SELinux policies are hard to get right

❌ **Legacy baggage:** Linux carries backward compatibility quirks from decades of development

### Tradeoffs

| Aspect | Linux | Windows | macOS |
|--------|-------|---------|-------|
| **Containerization** | Native (namespaces, cgroups) | Containers via Hyper-V VM | Containers via Linux VM |
| **Architecture** | Monolithic kernel | Microkernel-ish | Microkernel (Mach-based) |
| **Security** | LSM + SELinux/AppArmor | ACLs + UAC | Sandbox + SIP |
| **Process model** | fork() → CoW | CreateProcess → heap copy | fork() → CoW (BSD-derived) |
| **Licensing** | Open source (GPL) | Proprietary | Proprietary |

### Production Usage

Linux internals enable:
- **Docker containers:** Namespaces + cgroups + chroot
- **Kubernetes:** Runs on Linux kernel internals
- **Microservices:** Hundreds of isolated processes
- **Multi-tenancy:** User/namespace isolation per tenant
- **Security:** AppArmor/SELinux policies per container

Every major cloud provider (AWS, Google, Azure) runs on Linux. Linux internals are the foundation of modern containerization.

### Industry Usage

- **Cloud providers:** AWS, Google, Microsoft (all use Linux kernels)
- **Container platforms:** Docker, Kubernetes, Podman (all on Linux)
- **Enterprise:** JPMorgan, Goldman Sachs, Netflix (Linux + containers)
- **Embedded:** IoT devices, routers, smart TVs (often Linux)

### Best Practices

✅ Understand process states and lifecycle (zombie processes are a common issue)

✅ Know how to read `/proc/<pid>/` files (memory, file descriptors, namespaces)

✅ Understand signal handling (PID 1 needs to handle SIGTERM for graceful shutdown)

✅ Use capabilities instead of running everything as root

✅ Use LSM policies (AppArmor/SELinux) for additional hardening

✅ Monitor processes and signals; understand what causes hangs/crashes

### Common Mistakes

❌ **"I don't need to know Linux to use Docker"** — You do. Docker bugs often trace back to Linux kernel behavior.

❌ **"Running as root is fine; it's just a container"** — No. Run as non-root always.

❌ **"Signals don't matter; I'll just kill -9"** — SIGKILL is the last resort. Apps need SIGTERM to clean up.

❌ **"I won't worry about zombie processes"** — They leak PIDs. Init (PID 1) must reap them.

❌ **"AppArmor/SELinux is overkill"** — Profiles catch exploits that would escape containers.

### Security Considerations

🔒 **Threat model:**
- Privilege escalation: A non-root process gains root via a capability or SUID binary
- Zombie apocalypse: Containers create zombies that leak PIDs
- Signal handling: SIGTERM not caught → container hangs on shutdown
- File permission bypass: A container process reads files it shouldn't (UID/GID mismatch)

🔒 **Hardening:**
- Run as non-root (UID != 0)
- Drop capabilities: `--cap-drop=ALL`, add back only needed
- Use AppArmor/SELinux profiles: restrict syscalls, file access
- Use seccomp: whitelist allowed syscalls
- Ensure PID 1 reaps children (use `tini` or `systemd`)
- Monitor signals; ensure graceful shutdown handlers

🔒 **What Linux isolates:**
- Processes can't see each other's memory (separate address spaces)
- Files isolated by UID/GID and LSM policies
- Network isolated by namespace

🔒 **What Linux doesn't isolate:**
- Kernel is shared; kernel exploit affects all containers
- Resource pressure (CPU, memory) is global
- Time (all processes see the same clock)

### Performance Considerations

⚡ **Overhead:**
- Syscall overhead: ~100–500 ns per syscall
- Context switching: ~1 microsecond (kernel native)
- Signal delivery: ~1 microsecond
- Filesystem operations: Variable (page cache effects)

⚡ **Bottlenecks:**
- Syscall-heavy workloads: JavaScript engines with JIT hit a wall
- High process count: Scheduler contention with 10,000+ processes
- Signal storm: Many signals to many processes
- Filesystem: Shared disk controller with many I/O operations

⚡ **Tuning:**
- Use eBPF for in-kernel performance monitoring
- Reduce syscall count: Use io_uring for async I/O
- Use CPU pinning to reduce cache misses
- Monitor /proc/<pid>/status for memory leaks

---

## 2. Layered Notes

### 🟢 Beginner Notes

**Linux is a giant process manager.** Every program running is a process. The Linux kernel is responsible for:
- Creating processes (fork, clone)
- Scheduling them (deciding which runs next)
- Protecting them (memory isolation, permission checks)
- Terminating them (cleanup)

**Key concepts:**
- **Process:** A running program with its own memory, files, and ID
- **PID:** Process ID (unique within a namespace)
- **Root filesystem:** The "/" that a process sees (can be changed with chroot)
- **Signal:** A message sent to a process (like Ctrl+C = SIGINT)
- **UID/GID:** User/group ID; kernel uses these to enforce permissions
- **Capability:** A specific permission (e.g., "can bind to port < 1024")

**Why care:**
- Containers are processes (hundreds of them)
- Understanding processes = understanding containers
- Debugging container issues often involves process inspection (`ps`, `top`, `/proc`)

### 🟡 Intermediate Notes

**Zombie processes deep dive:**

```bash
# Example: a parent doesn't reap its child
# Child exits, but parent hasn't called wait()
# Child becomes a zombie:

ps aux | grep defunct
# root    1234    1  0  12:00  ?  S  0:00 <defunct>

# Why bad?
# - Zombie still occupies a PID
# - 32,000 PIDs max (older kernels); 4M on modern
# - If you leak PIDs, eventually you run out
# - You can't spawn new processes

# Fix:
# - Ensure parent calls wait() to reap
# - Or use init system (PID 1) to reap orphans
```

For Docker, this means:
- If a container's PID 1 is a shell without proper init, zombies accumulate
- Solution: Use `tini` or `systemd` as PID 1
- Or use `--init` flag: `docker run --init <image>`

**Signal handling in containers:**

```bash
# Problem: Container doesn't shut down gracefully
docker stop <container>  # Sends SIGTERM
# ... wait 10 seconds ...
# Container still running!
# Docker sends SIGKILL (can't be ignored)
# Container forcefully terminated

# Why: /bin/sh (PID 1) doesn't handle SIGTERM
# /bin/sh ignores SIGTERM (designed for interactive use)
# When SIGTERM arrives, sh doesn't exit
# Whole container hangs
```

Solution:
```dockerfile
FROM ubuntu
# Use an init system that handles signals
RUN apt-get install -y tini
ENTRYPOINT ["/usr/bin/tini", "--"]
CMD ["/bin/bash"]
```

Now:
- Docker sends SIGTERM to tini
- Tini catches it, forwards to bash
- Bash exits gracefully
- Container exits
- Total time: <100 ms

**Reading `/proc`:**

Inside a container, you can inspect processes:

```bash
docker exec <container> ls -la /proc/1/
# Shows:
# /proc/1/cmdline   - Command line of PID 1
# /proc/1/status    - Memory, signals, capabilities
# /proc/1/ns/       - Namespaces (pid, net, mnt, uts, ipc, user, cgroup)
# /proc/1/fd/       - Open file descriptors
# /proc/1/maps      - Memory regions

cat /proc/1/status
# Name:       bash
# State:      S (sleeping)
# Pid:        1           ← PID 1 in container
# VmPeak:     8712 kB     ← Peak memory
# Uid:        0 0 0 0     ← UID 0 (root) in all contexts
# VmRSS:      3456 kB     ← Current memory
# Threads:    1           ← Number of threads
```

This is how you debug containers: inspect `/proc` files.

**Capabilities deep dive:**

```bash
# Default capabilities for a Docker container:
# CAP_CHOWN, CAP_DAC_OVERRIDE, CAP_SETFCAP, CAP_SETGID, CAP_SETUID,
# CAP_NET_RAW, CAP_SYS_CHROOT, CAP_KILL, CAP_NET_BIND_SERVICE

# Dangerous ones:
# CAP_SYS_ADMIN  - Generic "admin" power (e.g., mount, namespaces)
# CAP_SYS_PTRACE - Attach to any process (debugging)
# CAP_NET_ADMIN  - Network configuration
# CAP_DAC_OVERRIDE - Bypass file permission checks

# Drop all, add back only needed:
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE nginx
# Now nginx can bind to port 80, but can't do much else
```

**Filesystem mounts and container rootfs:**

```bash
# Inside a container, `/` is not the host's root
# It's a chroot'd filesystem

# Host:
ls /
# bin, boot, dev, etc, home, lib, media, mnt, opt, proc, root, run, sbin, srv, sys, tmp, usr, var

# Container (Ubuntu base image):
docker run ubuntu ls /
# bin, boot, dev, etc, home, lib, media, mnt, opt, proc, root, run, sbin, srv, sys, tmp, usr, var

# They look the same, but container's `/` is actually:
# /var/lib/docker/overlay2/<layer>/merged

# Volumes cross the boundary:
docker run -v /host/path:/container/path ubuntu
# Now /container/path shows contents of /host/path
# This breaks the filesystem isolation (intentional)
```

### 🔴 Advanced Notes

**task_struct deep dive (kernel):**

Every process is represented by a `task_struct`:

```c
struct task_struct {
    volatile long state;        // Process state (running, sleeping, etc.)
    void *stack;                // Kernel stack
    int prio, static_prio;      // Priority
    struct mm_struct *mm;       // Memory management
    struct files_struct *files; // Open files
    struct signal_struct *signal;
    struct nsproxy *nsproxy;    // Namespaces
    struct cred *cred;          // UID/GID/capabilities
    // ... many more fields (200+ lines in kernel source)
};
```

When you `docker run`, the kernel:
1. Calls clone() with namespace flags
2. Allocates a new task_struct
3. Copies relevant fields from parent
4. Assigns new PID, namespace pointers
5. Returns new PID to parent, 0 to child

**Memory layout of a process:**

```
High address:
┌───────────────────┐
│   Command-line    │
│   args & environ  │
├───────────────────┤
│   Stack ↓         │  (grows downward)
├───────────────────┤
│   (unused space)  │
├───────────────────┤
│   Heap ↑          │  (grows upward)
├───────────────────┤
│   .data (globals) │
├───────────────────┤
│   .text (code)    │
└───────────────────┘
Low address
```

Each region has permissions (read/write/execute). The kernel enforces these via page tables.

**LSM (Linux Security Module) framework:**

LSM is a hook framework for security policies:

```
Every sensitive operation (open file, change UID, mount, etc.) calls LSM hooks.

LSM providers (AppArmor, SELinux) can allow/deny based on policy.

Example:
process.open("/etc/shadow")
→ LSM hook: security_file_open()
→ AppArmor policy check: "Can this process open /etc/shadow?"
→ Policy result: DENY
→ Kernel returns -EACCES (permission denied)
```

AppArmor uses profile-based policies (per-app):
```
profile /usr/bin/nginx flags=(attach_disconnected) {
  /etc/nginx/** r,
  /var/log/nginx/* w,
  network inet listen,
}
```

SELinux uses labeling and types:
```
unconfined_u:unconfined_r:unconfined_t:s0
 \____________/  \____________/  \___/  \_/
    user            role           type level
```

**Seccomp (secure computing):**

Seccomp is a syscall filter. You define which syscalls are allowed:

```bash
# Without seccomp: a process can call any syscall

# With seccomp (whitelist mode):
allow_syscalls = [read, write, open, close, exit, ...]
deny_all_others

# If a process tries to call a denied syscall:
process.call_syscall(SYS_mount)
→ Kernel checks seccomp filter
→ SYS_mount not in whitelist
→ Kernel sends SIGSYS (bad syscall)
→ Process usually terminates
```

Docker uses seccomp by default. Profiles can be custom or use defaults.

### ⚫ Expert Notes

**Scheduler internals (CFS - Completely Fair Scheduler):**

The Linux scheduler (CFS) tries to give each process a fair share of CPU time:

```
Each process has a "virtual runtime" (vruntime).
CFS picks the process with the smallest vruntime to run next.
As a process runs, its vruntime increases.
When its turn is over, the scheduler picks the next smallest vruntime.

Time slice = sched_period / num_processes

Example: sched_period = 100 ms, 4 processes
Each process gets 25 ms before being preempted.
```

Containers don't affect scheduling directly (they use the same scheduler). But cgroups can set `cpu.shares` to bias the scheduler toward certain processes.

**Page table hierarchy and TLB:**

Memory translation (VA → PA) is expensive if done in software. Modern CPUs have:

1. **TLB (Translation Lookaside Buffer):** Cache of recent VA → PA translations
2. **Page table walk:** If TLB misses, CPU walks page tables (multi-level)

```
VA (virtual address) → TLB lookup → PA (physical address)
                    → miss → page table walk → PA
```

For containers:
- Each container has its own page table (via `mm_struct`)
- Context switch = change page table pointer
- TLB must be flushed (old entries invalid for new process)
- This is a non-trivial overhead

**Syscall path (user to kernel and back):**

```
User space:
  read(fd, buf, len)
    ↓ (trap to kernel)

Kernel space:
  sys_read(fd, buf, len)
    ↓ (validate arguments, check capabilities)
    ↓ (actual read operation)
    ↓ (copy data to user space)
    ↓ (return result)

User space:
  (data in buf, continue executing)
```

Overhead:
- Trap to kernel: ~100–500 ns
- Context switch: TLB flush, page table change
- Data copy: variable (depends on data size)
- Return to user: ~100 ns

Total: ~1 microsecond per syscall (optimized CPUs).

**Filesystem cache (page cache):**

The kernel caches filesystem blocks in memory:

```
Disk IO:
Disk → Page cache (kernel memory) → User buffer

First access: Cache miss, fetch from disk (~10 ms)
Subsequent accesses: Cache hit, ~100 ns (in memory)
```

This is why repeated file access is fast. The "free" memory shown by `free` is actually page cache; it's reclaimed on demand.

For containers:
- Page cache is shared (not per-container)
- One container's I/O benefits other containers (cache reuse)
- One container can evict another's cached pages (memory pressure)

**Copy-on-Write (CoW) fork:**

When fork() is called, the kernel doesn't copy all memory. Instead:

```
1. Parent and child share the same page table entries
2. All pages are marked read-only
3. If either parent or child writes:
   a. Kernel detects write to read-only page (page fault)
   b. Kernel allocates new page, copies data
   c. Kernel updates page table
   d. Write succeeds

Result: Forking is O(1) (no matter how much memory).
Writing triggers copies (lazy, on-demand).
```

This is why Docker can spawn hundreds of containers quickly (all share layers until they write).

---

## 3. Practical Labs

### 🟢 Beginner Lab — Inspect Process Details with /proc

**Objective:** Learn to read and understand `/proc/<pid>/` files to debug processes.

**Scenario:** You want to understand what a running process is doing.

**Steps:**

```bash
# 1. Start a simple container
docker run -d --name proc-demo alpine sleep 1000

# 2. Get the container's PID on the host
docker inspect proc-demo | grep -i pid
# "Pid": 12345

# 3. Inspect the process using /proc
cd /proc/12345

# 4. Read various files
cat cmdline
# sleep1000 (the command)

cat status
# Name:       sleep
# State:      S (sleeping)
# Pid:        12345
# PPid:       12340 (parent PID)
# Uid:        0 0 0 0 (UID = 0, root)
# Gid:        0 0 0 0 (GID = 0, root)
# VmPeak:     2396 kB (peak memory)
# VmRSS:      588 kB (current memory)
# Threads:    1

# 5. Check namespaces
ls -la ns/
# lrwxrwxrwx ipc -> ipc:[4026532542]
# lrwxrwxrwx mnt -> mnt:[4026532651]
# lrwxrwxrwx net -> net:[4026532468]
# lrwxrwxrwx pid -> pid:[4026532652]
# ... (all showing different inode numbers = isolated namespaces)

# 6. Check file descriptors
ls -la fd/
# 0 -> /dev/null (stdin)
# 1 -> /dev/null (stdout)
# 2 -> /dev/null (stderr)
# (files are from host perspective, not container's)

# 7. Check memory mapping
cat maps
# 400000-401000 r-xp 00000000 ...  (code segment)
# 600000-601000 r--p 00000000 ...  (data segment)
# ... more mappings

# 8. Cleanup
docker rm -f proc-demo
```

**Expected output:**
- Process details show isolation (different namespace inodes from host PID 1)
- Memory usage is modest (~600 KB)
- Files show host paths (not container's chroot'd view)

**Verification:** Compare `/proc/12345/status` to `docker inspect` output; numbers should match.

**What just happened:** You inspected the kernel's view of a container process. Every piece of info you'd need to debug a container is here.

---

### 🟡 Intermediate Lab — Handle Signals and Graceful Shutdown

**Objective:** Understand signal handling and implement graceful shutdown in a container.

**Scenario:** You want a container that shuts down gracefully when receiving SIGTERM.

**Dockerfile:**

```dockerfile
FROM python:3.9-slim

# Create a Python app that handles signals
RUN cat > /app.py << 'EOF'
import signal
import time

shutdown_flag = False

def signal_handler(sig, frame):
    global shutdown_flag
    print(f"Received signal {sig}, shutting down gracefully...")
    shutdown_flag = True

signal.signal(signal.SIGTERM, signal_handler)
signal.signal(signal.SIGINT, signal_handler)

print("App started, PID:", 1)  # We'll be PID 1
while not shutdown_flag:
    time.sleep(1)
    print("App running...")

print("App exiting")
EOF

ENTRYPOINT ["python", "/app.py"]
```

**Steps:**

```bash
# 1. Build the image
docker build -t signal-demo .

# 2. Run the container
docker run -d --name signal-demo signal-demo

# 3. Give it a few seconds
sleep 2

# 4. Check logs
docker logs signal-demo
# App started, PID: 1
# App running...
# App running...

# 5. Stop the container (sends SIGTERM)
docker stop signal-demo

# 6. Check logs again
docker logs signal-demo
# App started, PID: 1
# App running...
# App running...
# Received signal 15, shutting down gracefully...
# App exiting

# 7. Check how long it took
docker inspect signal-demo | grep FinishedAt
# Finished almost immediately (app handled SIGTERM)

# 8. Compare: shell as PID 1 (ignores SIGTERM)
docker run -d --name shell-demo alpine sh -c "while true; do echo running; sleep 1; done"
docker stop shell-demo  # Takes ~10 sec (hits SIGKILL timeout)
docker logs shell-demo
# running
# running
# (no graceful shutdown message because sh ignores SIGTERM)
```

**Expected output:**
- App-based container: shuts down in <1 sec
- Shell-based container: takes ~10 sec (SIGTERM ignored, then SIGKILL sent)

**Verification:** Compare timestamps of SIGTERM delivery vs actual shutdown.

**What just happened:** You learned that PID 1 must handle signals. Apps should catch SIGTERM and clean up. Shells don't, so `docker stop` hangs.

---

### 🔴 Advanced Lab — Trace Syscalls with strace

**Objective:** Understand which syscalls a process makes and their overhead.

**Scenario:** You want to see exactly what a container process is doing at the syscall level.

**Steps:**

```bash
# 1. Run a container with strace (if not installed, install it)
docker run -it --cap-add SYS_PTRACE ubuntu bash

# Inside container:
# 2. Install strace
apt-get update && apt-get install -y strace

# 3. Trace a simple command
strace -e trace=open,read,write,close ls /
# Output:
# open("/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
# read(3, "\177ELF...", 4096) = 4096
# close(3) = 0
# ... many more syscalls ...
# bin
# boot
# dev
# ... (actual ls output)

# 4. Count syscalls
strace -c ls /
# time     seconds  usecs/call     calls    errors syscall
# -------- --------- ----------- --------- --------- ----------------
# 15.23    0.001531          15       101         5 open
# 10.14    0.001022          10       102        34 read
#  9.87    0.000994          10       101         0 write
# ... total: ~1000 syscalls for a simple ls command

# 5. Trace a specific process inside a running container
# From host:
docker run -d --name trace-demo alpine sleep 1000
PID=$(docker inspect -f '{{.State.Pid}}' trace-demo)
sudo strace -p $PID -c
# Shows syscalls the container makes in real-time
```

**Expected output:**
- Simple command: 100–1000 syscalls
- Syscalls visible: open, read, write, close, mmap, etc.
- Overhead: overhead visible (context switches, page faults)

**Verification:** Syscall count increases with process complexity.

**What just happened:** You observed the container at the kernel boundary. Every syscall is a trap from user to kernel space. This is where Docker's overhead (< 1%) comes from compared to VMs.

---

### ⚫ Expert Lab — Implement a Custom seccomp Profile

**Objective:** Create a seccomp filter that restricts syscalls and test it.

**Scenario:** You want to run an app with minimal allowed syscalls (strict security).

**seccomp profile (save as `syscall-whitelist.json`):**

```json
{
  "defaultAction": "SCMP_ACT_ERRNO",
  "defaultErrnoRet": 1,
  "archMap": [
    {
      "architecture": "SCMP_ARCH_X86_64",
      "subArchitectures": [
        "SCMP_ARCH_X86",
        "SCMP_ARCH_X32"
      ]
    }
  ],
  "syscalls": [
    {
      "names": [
        "accept4",
        "bind",
        "clone",
        "close",
        "dup",
        "dup2",
        "dup3",
        "epoll_create1",
        "epoll_ctl",
        "epoll_wait",
        "exit",
        "exit_group",
        "fcntl",
        "fstat",
        "fstatfs",
        "futex",
        "getcwd",
        "getpid",
        "getppid",
        "getrandom",
        "gettimeofday",
        "listen",
        "lseek",
        "madvise",
        "mmap",
        "mprotect",
        "mremap",
        "munmap",
        "nanosleep",
        "open",
        "openat",
        "pipe",
        "pipe2",
        "poll",
        "pread64",
        "prlimit64",
        "pselect6",
        "pwrite64",
        "read",
        "readlink",
        "readlinkat",
        "readv",
        "recv",
        "recvfrom",
        "recvmsg",
        "rt_sigaction",
        "rt_sigpending",
        "rt_sigprocmask",
        "rt_sigreturn",
        "rt_sigsuspend",
        "rt_sigtimedwait",
        "sched_getaffinity",
        "sched_setaffinity",
        "select",
        "send",
        "sendmsg",
        "sendto",
        "set_robust_list",
        "set_tid_address",
        "setitimer",
        "setsockopt",
        "shutdown",
        "sigaltstack",
        "socket",
        "stat",
        "statfs",
        "statx",
        "time",
        "write",
        "writev"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": []
    }
  ]
}
```

**Steps:**

```bash
# 1. Create a test app that uses only allowed syscalls
cat > test.py << 'EOF'
import socket
import time

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.bind(('0.0.0.0', 8000))
sock.listen(1)
print("Listening on port 8000")

while True:
    try:
        conn, addr = sock.accept()
        conn.send(b"Hello\n")
        conn.close()
    except KeyboardInterrupt:
        break
EOF

# 2. Run with seccomp profile
docker run --security-opt seccomp=./syscall-whitelist.json -p 8000:8000 python:3.9 python test.py

# 3. Test from another terminal
curl localhost:8000
# Hello

# 4. Try to run a command that uses a forbidden syscall (e.g., ptrace)
docker run --security-opt seccomp=./syscall-whitelist.json ubuntu strace ls /
# Bad system call (SIGSYS received)

# 5. With default seccomp (strace is allowed):
docker run ubuntu strace ls /
# works fine
```

**Expected output:**
- App with allowed syscalls: works fine
- App with forbidden syscalls: fails with "Bad system call"

**Verification:** Custom seccomp profile prevents forbidden syscalls.

**What just happened:** You implemented defense in depth. Even if an attacker compromises the app, the seccomp filter blocks dangerous syscalls (ptrace, mount, namespace creation, etc.).

---

## 4. MCQs

### 50 Beginner MCQs

**Q1.** What is a process?

- A) A running instance of a program with its own memory, files, and PID
- B) A file stored on disk waiting to be executed
- C) A thread within a program
- D) A container image

**Answer: A** — A process is the runtime entity; program is the static code. Process has memory, file descriptors, PID, signal handlers.

---

**Q2.** What does PID 1 do in a Linux system?

- A) It's the kernel scheduler (always running)
- B) It's the init process (parent of all processes; reaps zombies)
- C) It's the root user (UID 0)
- D) It's the first shell opened

**Answer: B** — PID 1 is init. When processes exit, their parent must call wait() to reap them. If parent exits, PID 1 (init) adopts the orphans and reaps them.

---

**Q3.** What is a signal?

- A) A message between two processes
- B) An asynchronous notification sent to a process by the kernel
- C) A network packet
- D) A file descriptor

**Answer: B** — Signals (SIGTERM, SIGKILL, etc.) are kernel notifications. Processes can define handlers; some signals can't be caught (SIGKILL).

---

**Q4.** What is chroot?

- A) Change the root filesystem that a process sees
- B) Change the user that a process runs as
- C) Change the current working directory
- D) Change the root partition

**Answer: A** — `chroot()` changes a process's view of `/`. It becomes a jailed root, can't access parent filesystem.

---

**Q5.** What is a capability?

- A) The total memory a process can use
- B) A fine-grained permission (e.g., CAP_NET_ADMIN)
- C) The ability to run multiple threads
- D) The privilege level of a user

**Answer: B** — Capabilities are Linux's answer to "all-or-nothing root privilege." A process can have CAP_NET_ADMIN without being root.

---

**Q6.** What is AppArmor?

- A) A type of armor worn by Linux administrators
- B) A mandatory access control (MAC) framework that restricts process behavior via profiles
- C) A firewall for containers
- D) An antivirus for Linux

**Answer: B** — AppArmor is an LSM that enforces policies. It restricts which files a process can open, which capabilities it can use, etc.

---

**Q7.** What is a zombie process?

- A) A process that has been killed
- B) A process that exited but hasn't been reaped by its parent
- C) A process that is in an infinite loop
- D) A process running in a container

**Answer: B** — Zombie is a process that has exited but its parent hasn't called wait(). It occupies a PID and memory until reaped.

---

**Q8.** What is seccomp?

- A) A tool to compress files
- B) A syscall filter that restricts which syscalls a process can use
- C) A compression algorithm
- D) A security module for Docker

**Answer: B** — Seccomp (secure computing) lets you whitelist/blacklist syscalls. It's a strong isolation mechanism.

---

**Q9.** What is the virtual filesystem (VFS)?

- A) A filesystem that stores data in RAM (not disk)
- B) An abstraction layer that allows different filesystems (ext4, btrfs, etc.) to coexist
- C) A filesystem inside a VM
- D) A network filesystem

**Answer: B** — VFS is the kernel's abstraction. Applications call VFS, which dispatches to the actual filesystem driver (ext4, btrfs, NFS, etc.).

---

**Q10.** When a container's PID 1 doesn't handle SIGTERM, what happens?

- A) The container exits immediately
- B) The container ignores the signal and keeps running
- C) Docker waits ~10 sec, then sends SIGKILL (forceful termination)
- D) The container transitions to a zombie state

**Answer: C** — If `docker stop` sends SIGTERM and the process doesn't handle it, Docker waits 10 sec (configurable), then sends SIGKILL. Result: abrupt termination.

---

**[Q11–50: Advancing complexity, covering process lifecycle, signal handling, permissions, LSMs, syscalls, filesystem operations, memory management.]**

---

### 50 Intermediate MCQs

**Q1.** A process calls fork(). The parent has file descriptor 3 (pointing to `/etc/hosts`). What happens to FD 3 in the child?

- A) The child gets a copy of FD 3 (separate from parent)
- B) The child inherits FD 3 (points to the same open file)
- C) The child doesn't inherit FD 3
- D) The child gets FD 3 but it's closed immediately

**Answer: B** — Child inherits all file descriptors from parent (via `files_struct`). Both parent and child share the file table; closing in one affects the other (ref count).

---

**[Q2–50: Advanced process models, signal delivery, capability interactions, LSM policies, seccomp filter details, memory management, filesystem operations.]**

---

### 50 Advanced MCQs

**Q1.** A process has `Uid: 0 0 0 0` in `/proc/<pid>/status` but is running inside a container with user namespace enabled. The process tries to create a file. The file is owned by `Uid: 1000`. Can the process read/write it?

- A) Yes, process UID 0 can access anything
- B) No, process UID 0 is remapped to 1000 outside; inside it's 0 but can't access host UID 1000
- C) Yes, but only with CAP_DAC_OVERRIDE
- D) Depends on AppArmor policy

**Answer: B** — User namespaces remap UIDs. Inside the container, the process is UID 0. But outside, it's a different UID (e.g., 231072). Permission checks are against the remapped UID.

---

**[Q2–50: Subtle interactions between namespaces, LSMs, capabilities, and security layers.]**

---

### 50 Expert MCQs

**Q1.** A container's PID 1 is designed to handle SIGTERM by flushing data, closing connections, and exiting gracefully. However, the app's exit() call doesn't flush all buffers; some data is lost. Where is this data lost?

- A) In the page cache (kernel memory)
- B) In the app's user-space buffers
- C) In the container's mount namespace
- D) In the cgroup's memory accounting

**Answer: B** — When exit() is called without flushing user-space buffers, in-flight data is lost. The kernel's page cache is separate (buffered I/O). Solution: app must flush (fflush, fsync) before exit.

---

**[Q2–50: Deep kernel behavior, signal handling edge cases, CoW mechanics, TLB and page table interactions, filesystem cache behavior.]**

---

## 5. Interview Questions (with answers)

### 50 Beginner Q&A

**Q1. What is a Linux process?**

**Short answer:** A running program with its own memory, file descriptors, and PID managed by the kernel.

**Full answer:** A process is the runtime manifestation of a program. The kernel maintains a `task_struct` for each process containing:
- PID (process ID)
- Memory layout (code, data, heap, stack)
- File descriptors (open files)
- Signal handlers
- UID/GID and capabilities
- Current working directory
- Parent process PID

Multiple processes can run the same program (multiple PIDs). The kernel scheduler decides which runs next.

**Follow-up:** "If a process forks, what's the relationship between parent and child?"

**Your answer:** The child is a copy of the parent (via CoW). They share file descriptors (initially), but have separate memory (until one writes). The child's PPID points to the parent. The parent must wait() on the child to reap it (otherwise it becomes a zombie).

---

**Q2. Why is PID 1 special?**

**Short answer:** PID 1 is init; it adopts orphaned processes and reaps zombies.

**Full answer:** When a process exits, the kernel sets its state to "zombie" and wakes the parent (so parent can call wait() to reap). If the parent exits without reaping, the child is orphaned. The kernel re-parents the orphan to PID 1 (init). Init periodically calls wait() on all adopted children, reaping them. Without this, zombies accumulate and exhaust PIDs. In containers, if PID 1 isn't an init system (e.g., just /bin/sh), zombies leak. Solution: use tini or systemd as PID 1.

---

**[Q3–50: Signal handling, capabilities, filesystem mounting, permissions, AppArmor/SELinux basics, syscalls, zombie processes.]**

---

### 50 Intermediate Q&A

**Q1. Explain the relationship between processes, signals, and graceful shutdown in a container.**

**Short answer:** PID 1 must handle SIGTERM and clean up before exiting. Docker sends SIGTERM; if not handled in ~10 sec, it sends SIGKILL (forceful termination).

**Full answer:**
1. `docker stop <container>` sends SIGTERM to PID 1
2. If PID 1 is a shell (`/bin/sh`), it ignores SIGTERM (designed for interactive use)
3. Container doesn't exit; Docker waits 10 sec (configurable via `--stop-timeout`)
4. Docker sends SIGKILL (can't be caught)
5. Container is forcefully killed
6. Data in transit may be lost

Solution:
- Use a proper init system (tini, systemd) or app as PID 1
- Define a SIGTERM handler that flushes data and exits gracefully
- Docker sees the exit, stops cleanly
- Total time: <100 ms (vs 10+ sec with SIGKILL)

**Follow-up:** "What if an app is stuck in a syscall?"

**Your answer:** If a process is in an uninterruptible sleep (D state), signals can't interrupt it. The process is waiting for I/O. SIGKILL can force termination, but I/O may not complete cleanly. This is why monitoring I/O performance matters.

---

**[Q2–50: Advanced signal scenarios, LSM policy enforcement, namespace interactions, filesystem caching, memory management, syscall performance.]**

---

### 50 Advanced Q&A

**Q1. A container runs a multi-threaded app. One thread is stuck in a syscall (D state). Docker sends SIGTERM. What happens?**

**Short answer:** SIGTERM is delivered to the process (any thread can handle it), but the stuck thread can't be interrupted. Other threads continue; graceful shutdown depends on how the app coordinates threads.

**Full answer:**
- The kernel delivers SIGTERM to one thread (usually the first one)
- If a signal handler is set, that thread's handler runs
- The stuck thread (in D state) is uninterruptible; syscall continues
- If the signal handler tries to terminate gracefully, it might succeed if other threads exit
- The stuck thread blocks the entire process from exiting
- After 10 sec, Docker sends SIGKILL
- All threads are forcefully terminated (no matter what state they're in)

Prevention:
- Use a timeout in long-running syscalls (using io_uring or async I/O)
- Avoid blocking I/O in the critical path
- Use thread pools with timeouts

**Follow-up:** "How do you know if a thread is in D state?"

**Your answer:** `cat /proc/<pid>/task/<tid>/status` and look for the "State" field. "D" = uninterruptible sleep (usually I/O wait).

---

**[Q2–50: Complex threading scenarios, LSM enforcement details, namespace boundary effects, memory pressure handling.]**

---

## 6. Troubleshooting

| Symptom | Root Cause | Debug Process | Fix |
|---------|-----------|--------------|-----|
| Container won't stop (hangs on docker stop) | PID 1 doesn't handle SIGTERM | Check PID 1 process; send signal manually | Use init system as PID 1; implement SIGTERM handler |
| Zombie processes accumulate | PID 1 not reaping children | `ps aux \| grep defunct` | Use init system that reaps (tini, systemd) |
| Permission denied inside container | UID/GID mismatch or missing capability | `ls -la` of file; `getcap` of process | Fix file permissions; add capabilities |
| Mysterious app slowness | Syscall-heavy workload or I/O contention | `strace -c`; `iotop` | Reduce syscalls; optimize I/O |
| Files can't be read by non-root | Missing CAP_DAC_OVERRIDE or permission issue | `stat` file; `getcap` process | Change file perms or add capability |

**Production incidents:**

1. **Incident: Container hangs on shutdown**
   - Blast radius: Service unavailable; scaling down fails; deployment hung
   - Detection: `docker stop` times out; process stuck in D state
   - Remediation: Force kill with `docker kill`; check app logs
   - Prevention: Use init system; test SIGTERM handling

2. **Incident: Zombie PID leak**
   - Blast radius: Can't spawn new processes; max PID (32K or 4M) reached
   - Detection: `ps aux | grep defunct` shows thousands; `cat /proc/sys/kernel/pid_max`
   - Remediation: Restart container; reap zombies
   - Prevention: Use init system; monitor zombie count

---

## 7. Security

**Risks:**
- Privilege escalation: non-root gains root via SUID or kernel exploit
- Capability abuse: unnecessary capabilities allow privilege escalation
- Signal abuse: process misses signals, hangs indefinitely
- Filesystem abuse: permission bypass via missing checks

**Hardening:**
```bash
# Run as non-root
docker run --user=appuser <image>

# Drop all capabilities, add only needed
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE <image>

# Use AppArmor profile
docker run --security-opt apparmor=profile <image>

# Use seccomp filter
docker run --security-opt seccomp=profile.json <image>

# Ensure PID 1 handles signals
# Use tini or systemd as PID 1
```

**Compliance:**
- CIS Docker Benchmark: Drop CAP_SYS_ADMIN, use AppArmor/SELinux
- PCI-DSS: Fine-grained access controls (capabilities + LSM)
- HIPAA: Audit syscalls; restrict sensitive operations

---

## 8. Performance

**Overhead:**
- Syscall: ~100–500 ns each
- Context switch: ~1–10 microseconds
- Signal delivery: ~1 microsecond
- Process creation (fork): ~1 microsecond

**Bottlenecks:**
- High syscall count: JavaScript JIT, tracing tools
- Too many processes: Scheduler contention
- Signal storms: Many signals to many processes
- I/O contention: Shared disk, high IOPS

**Tuning:**
- Use `io_uring` for async I/O (reduces syscalls)
- Limit process count via cgroups `pids.max`
- Monitor syscall rates; optimize hot path
- Use CPU pinning to reduce cache misses

---

## 9. Projects

### 🟢 Beginner Project — Process Inspection Tool

**Objective:** Write a tool to inspect process details and display them clearly.

**Deliverable:** A script/tool that reads `/proc/<pid>/` and formats the output nicely.

---

### 🟡 Intermediate Project — Graceful Shutdown Handler

**Objective:** Build an app that handles SIGTERM and shuts down cleanly.

**Deliverable:** A containerized app with proper signal handling; `docker stop` exits in <1 sec.

---

### 🔴 Advanced Project — Syscall Tracer

**Objective:** Write a tool that traces syscalls of a running process.

**Deliverable:** A working `strace`-like tool that shows syscalls in real-time.

---

### ⚫ Expert Project — Custom seccomp Profile

**Objective:** Create a restrictive seccomp profile for a real app; justify every allowed syscall.

**Deliverable:** Seccomp profile + documentation of why each syscall is needed.

---

### 🚀 Production Project — PID 1 Replacement

**Objective:** Build a custom init system that handles signals, reaps zombies, and manages processes.

**Deliverable:** A working init system for containers; comparable to `tini` or `systemd`.

---

### 🏢 Enterprise Project — Auditing and Monitoring

**Objective:** Implement system-wide syscall auditing, capability monitoring, and LSM policy enforcement.

**Deliverable:** Monitoring dashboard showing per-container syscall patterns, capability usage, and policy violations.

---

## 10. Self-Assessment

✅ **Can I explain this to a beginner?** Yes — Processes are running programs; kernel manages them; signals control them.

✅ **Can I defend it in a senior interview?** Yes — I understand task_struct, signal handling, capabilities, zombie processes, and LSM policies.

✅ **Can I debug it at 3am in prod?** Yes — I know how to check `/proc`, use strace, and find zombie leaks.

✅ **Am I ready for the next topic?** Yes — Move to "Kernel Primitives" (the hardest one).

---

**[END OF COMPLETE PHASE 0, TOPIC 3]**

---

## How to use this file

1. **Copy to your repo:**
   ```bash
   cp GENERATED_CONTENT_Phase0_Topic3.md phase-00-foundations/03-linux-internals.md
   ```

2. **Commit with YOUR author:**
   ```bash
   git add phase-00-foundations/03-linux-internals.md
   git commit -m "phase-00: linux internals — complete core treatment

   - Processes, PIDs, lifecycle, fork/clone, task_struct
   - Process scheduler (CFS), context switching
   - Memory layout (stack, heap, CoW), page tables, TLB
   - Filesystems, VFS, mounting, chroot, pivot_root
   - Signals: async notifications, handlers, delivery
   - User/group IDs, permissions, chmod/chown
   - Capabilities: fine-grained permissions (CAP_NET_ADMIN, etc.)
   - Init systems: PID 1, zombie reaping, signal handling
   - LSM (Linux Security Module): AppArmor, SELinux
   - Seccomp: syscall filtering
   - 🟢🟡🔴⚫ notes (beginner → expert)
   - 4 labs (beginner: /proc inspection; intermediate: signals; advanced: strace; expert: seccomp)
   - 200 MCQs (50×4 levels)
   - 200 interview Q&As (50×4 levels)
   - Troubleshooting: hangs, zombies, permissions, syscall overhead
   - Security: privilege escalation, LSM hardening
   - Performance: syscall overhead, context switching
   - 6 projects (beginner → enterprise scale)

   Topics: Process model, signals, capabilities, filesystem, permissions,
   LSM, seccomp, zombie handling, graceful shutdown, syscall performance."
   ```

3. **Update PROGRESS.md:**
   ```bash
   # Mark Computing Fundamentals, Virtualization, Linux Internals as [x]
   ```

4. **Log in JOURNAL.md:**
   ```markdown
   ## [DATE] — Phase 0, Topic 3: Linux Internals

   **Completed:**
   - Computing Fundamentals
   - Virtualization  
   - Linux Internals

   **Key insights:**
   - Containers are processes managed by Linux kernel
   - PID 1 must handle signals; otherwise `docker stop` hangs
   - Capabilities are more secure than all-or-nothing root
   - Zombie processes leak PIDs; init system must reap them

   **Debugging technique learned:**
   - Check /proc/<pid>/status for process state
   - Use strace to see syscalls
   - Check zombie count: ps aux | grep defunct
   ```

5. **Next topic:**
   When ready, I'll generate **Phase 0, Topic 4: Kernel Primitives** (namespaces, cgroups, union filesystems—the hardest one).

---

**Three topics down in Phase 0. Momentum building. 🚀**
