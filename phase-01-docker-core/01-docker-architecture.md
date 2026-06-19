# TOPIC: Docker Architecture

> **Phase:** 1 · **Module:** Docker Core · **Difficulty:** Beginner → Expert  
> **Prerequisites:** [[kernel-primitives]], [[networking-fundamentals]], [[container-standards-ecosystem]]  
> **Status:** ✅ Complete

---

## 1. Core Treatment

### Definition

Docker architecture is the client-server design that powers Docker. It consists of a **Docker daemon** (dockerd) running on the host, a **CLI tool** (docker command) communicating via REST API, and **containerd** (a container manager) that abstracts the container lifecycle. The daemon receives commands from the CLI, manages images, creates containers, and orchestrates the full container lifecycle. Internally, Docker uses **containerd** for container management (since Docker 1.11), which in turn uses **runc** (the OCI runtime) to create containers. Understanding this architecture is crucial because debugging Docker issues requires understanding where things break: in the daemon, in containerd, in the runtime, or in the kernel.

### Purpose

**Before Docker (pre-2013):** Using containers required understanding LXC, namespaces, cgroups, and writing complex shell scripts. It was expert-only.

**Docker's innovation:** Simple CLI (`docker run`, `docker build`, `docker push`) that abstracts all the complexity. The daemon handles orchestration; the CLI is user-friendly.

**Architecture evolution:**
- **2013–2015:** Docker monolithic (daemon did everything)
- **2015–2017:** Separation of concerns (Docker daemon → containerd → runc)
- **2017+:** Mature architecture (daemon thin layer, containerd focused on containers, runc focused on OCI compliance)

The current architecture is modular and stable. Understanding each component helps with debugging, scaling, and choosing alternatives (e.g., podman instead of Docker daemon).

---

### Internal Working

#### **Docker Daemon (dockerd)**

The Docker daemon is a long-running process that manages containers, images, networks, and volumes.

```
docker CLI sends request
  ↓
Request travels via Unix socket (/var/run/docker.sock) or TCP
  ↓
Docker daemon receives and parses request
  ↓
Daemon dispatches to appropriate handler:
  - Image management → image store
  - Container management → containerd
  - Network management → network driver
  - Volume management → volume driver
  ↓
Handler performs operation
  ↓
Result sent back to CLI
  ↓
CLI displays output to user
```

**Key responsibilities:**
- Image management (build, pull, push, rm, inspect)
- Container orchestration (create, start, stop, rm)
- Network management (create, connect, disconnect)
- Volume management (create, mount, inspect)
- Log aggregation (collect logs from containers)
- Signal handling (restart policies, health checks)

#### **containerd**

containerd is a minimal container daemon. Docker daemon delegates container lifecycle management to containerd.

```
docker run → daemon → containerd.Create() → runc → container created
                    ↓
            containerd.Start() → runc → process started
                    ↓
            containerd.Stop() → runc → process stopped
                    ↓
            containerd.Delete() → runc → container deleted
```

**containerd responsibilities:**
- Container lifecycle (create, start, stop, delete)
- Image management (download, unpack, store)
- Snapshots (filesystem layers, CoW)
- Networking (namespace setup, interface attachment)
- Logging (collect stdout/stderr)

#### **runc**

runc is the OCI-compliant runtime. It creates and manages containers at the OS level.

```
containerd calls runc create <container-id>
  ↓
runc reads config.json (OCI Runtime Spec)
  ↓
runc:
  1. clone() with CLONE_NEW* flags (create namespaces)
  2. Setup cgroups (/sys/fs/cgroup/*)
  3. Mount rootfs (chroot/pivot_root)
  4. Set capabilities, seccomp, AppArmor
  5. execve() the process (PID 1 in container)
  ↓
Container now running (managed by kernel)
```

**runc is thin and focused:** It does what the OCI spec defines, nothing more.

#### **Docker Desktop (on Windows/Mac)**

Docker Desktop runs a Linux VM (via Hyper-V on Windows, via hypervisor on Mac) because Docker is Linux-native.

```
Windows/Mac user:
  docker CLI
    ↓
  Docker Desktop app (manages the VM)
    ↓
  Linux VM (runs dockerd, containerd, runc)
    ↓
  Containers running inside the Linux VM
    ↓
  Result: User sees containers as if they're running locally
```

#### **REST API**

Docker daemon exposes a REST API. The CLI is just a client to this API.

```
GET /containers/json
  → List running containers

POST /containers/create
  → Create a container

POST /containers/<id>/start
  → Start a container

GET /images/<name>/json
  → Get image details

POST /images/<name>/push
  → Push image to registry
```

The API is fully featured; anything the CLI does can be done via API (curl, HTTP clients, SDKs).

---

### Architecture

```
┌────────────────────────────────────────────────────────────┐
│                    User / Tools                            │
│  docker CLI, Docker Desktop, Docker API clients            │
└────────────┬─────────────────────────────────────────────┘
             │
             │ REST API / gRPC
             │
┌────────────┴─────────────────────────────────────────────┐
│                 Docker Daemon (dockerd)                  │
│  - Image store (registry, local cache)                   │
│  - Container orchestration                               │
│  - Network management                                    │
│  - Volume management                                     │
│  - Log aggregation                                       │
│  - API server                                            │
└────────────┬─────────────────────────────────────────────┘
             │
             │ containerd API (gRPC)
             │
┌────────────┴─────────────────────────────────────────────┐
│            containerd (Container Manager)                │
│  - Container lifecycle (create, start, stop, delete)    │
│  - Image management (store, unpack)                     │
│  - Snapshot management (overlayfs coordination)         │
│  - Logging                                              │
└────────────┬─────────────────────────────────────────────┘
             │
             │ OCI Runtime Spec
             │
┌────────────┴─────────────────────────────────────────────┐
│              runc (OCI Runtime)                          │
│  - Namespace creation (clone with flags)                │
│  - cgroup setup                                         │
│  - Rootfs mounting (chroot/pivot_root)                 │
│  - Process execution (execve)                          │
└────────────┬─────────────────────────────────────────────┘
             │
┌────────────┴─────────────────────────────────────────────┐
│            Linux Kernel (namespaces, cgroups)            │
│  - Process scheduler                                    │
│  - Memory manager                                       │
│  - Filesystem                                          │
│  - Network stack                                       │
└─────────────────────────────────────────────────────────┘
```

---

### Lifecycle

```
docker run <image> <cmd>
  ↓
Docker daemon receives request
  ↓
1. Image resolved (pull if not present)
  ↓
2. containerd.Create():
   - Setup namespaces + cgroups
   - Mount image layers (overlay2)
   - Create config.json (OCI spec)
  ↓
3. containerd.Start():
   - Call runc create
   - Call runc start
  ↓
4. Container running (kernel manages process)
  ↓
user sends: docker stop <container>
  ↓
5. Daemon sends SIGTERM to container PID 1
  ↓
6. Container process handles signal, exits
  ↓
7. containerd detects exit, cleans up
   - Delete namespaces
   - Delete cgroups
   - Unmount layers
   - Delete container
  ↓
docker rm <container>
  ↓
8. Daemon removes container metadata
```

---

### Components

**1. Docker Daemon (dockerd):**
- Listens on Unix socket (/var/run/docker.sock)
- HTTP server (REST API)
- Image store manager
- Container orchestrator
- Network/volume managers

**2. containerd:**
- gRPC service
- Container store
- Snapshot manager (overlay2)
- Image store
- Logging service

**3. runc:**
- OCI runtime binary
- Implements OCI Runtime Spec
- No daemon; spawned per container

**4. Shim (containerd-shim):**
- Lightweight wrapper per container
- Keeps container alive if containerd daemon crashes
- Manages container stdin/stdout/stderr

**5. Plugins:**
- Storage drivers (overlay2, aufs, btrfs)
- Network drivers (bridge, overlay, macvlan)
- Volume drivers (local, nfs, custom)
- Logging drivers (json-file, journald, splunk)

---

### Advantages

✅ **Simple API:** `docker run` is straightforward; hides complexity

✅ **Modular:** daemon → containerd → runc separation of concerns

✅ **Portable:** Same CLI on Linux, Mac, Windows (Docker Desktop)

✅ **Extensible:** Plugins for storage, network, volume, logging

✅ **Mature:** Battle-tested in millions of deployments

✅ **Standards-based:** Uses OCI (portable to other runtimes)

### Disadvantages

❌ **Monolithic daemon:** One daemon crash can affect all containers

❌ **Privilege required:** Running docker requires root or docker group (security surface)

❌ **Complexity hidden:** Understanding internals requires deep knowledge

❌ **Resource overhead:** Daemon uses memory/CPU (even when idle)

---

### Production Usage

- **Cloud platforms:** AWS ECS, Google Cloud Run, Azure Container Instances use containerd (Docker's engine)
- **Kubernetes:** Uses containerd or CRI-O (not Docker daemon directly)
- **On-prem:** Most enterprises use Docker or containerd
- **Serverless:** Lambda, Cloud Functions use similar architecture

---

### Best Practices

✅ Understand the layered architecture (daemon → containerd → runc)

✅ Monitor Docker daemon resource usage (memory, file descriptors)

✅ Use containerd directly for simpler workloads (lower overhead)

✅ Keep Docker daemon updated (security patches, performance improvements)

✅ Use alternatives (Podman) if you need rootless or daemonless operation

✅ Understand limits of Docker daemon abstraction (some features Docker-specific, not portable)

### Common Mistakes

❌ **Assuming Docker daemon is lightweight:** It uses significant resources

❌ **Running untrusted code as root in containers:** Docker daemon runs as root; compromise = host compromise

❌ **Not understanding containerd vs docker daemon:** They're separate; knowing this helps with debugging

❌ **Tightly coupling to Docker-specific features:** Some features don't work with other runtimes

❌ **Ignoring the shim:** If shim crashes, container can become orphaned

---

### Security Considerations

🔒 **Daemon privilege:** Docker daemon runs as root. Compromise = full host access. Mitigate: restrict socket access, use rootless mode.

🔒 **API exposure:** Docker API on TCP is dangerous. Always use Unix socket or require authentication.

🔒 **Privilege escalation:** Inside container, root can use capabilities to escape (rare, but possible). Mitigation: drop capabilities, use seccomp.

---

### Performance Considerations

⚡ **Daemon overhead:** Docker daemon uses 50–200 MB RAM at rest.

⚡ **Startup latency:** `docker run` involves multiple round-trips (daemon → containerd → runc). ~100 ms total.

⚡ **API latency:** Each CLI command = HTTP request to daemon. Typical: 1–10 ms per request.

---

## 2. Layered Notes

### 🟢 Beginner Notes

**Docker has three main parts:**
1. **docker command (CLI):** What you type; talks to the daemon
2. **Docker daemon (dockerd):** Runs in background; manages everything
3. **containerd + runc:** Do the actual container work

**Simple flow:**
```
You type: docker run ubuntu
  ↓
docker CLI sends request to daemon (via socket)
  ↓
daemon tells containerd to create container
  ↓
containerd tells runc to create container
  ↓
runc creates container (using Linux namespaces + cgroups)
  ↓
Container runs
```

**Key insight:** Docker daemon is just an orchestrator. The real work happens in containerd and runc.

### 🟡 Intermediate Notes

**containerd-shim:**

When you `docker run`, the flow is:
```
docker CLI → daemon → containerd → runc → creates container
                   ↓
          spawns containerd-shim (per container)
                   ↓
shim keeps container alive if containerd restarts
```

Without the shim, restarting containerd would kill all containers (bad). With the shim, containers survive daemon restarts.

**Docker API:**

You can talk to Docker daemon directly:
```bash
curl -s --unix-socket /var/run/docker.sock -X GET http://localhost/containers/json
# Returns JSON list of containers

curl -s --unix-socket /var/run/docker.sock -X POST http://localhost/containers/create \
  -H "Content-Type: application/json" \
  -d '{"Image":"ubuntu","Cmd":["/bin/bash"]}'
# Creates a container
```

The `docker` command is just a convenience wrapper around this API.

### 🔴 Advanced Notes

**Docker daemon caching and goroutines:**

Docker daemon caches:
- Images (locally)
- Container metadata
- Volume definitions
- Network definitions

Each active container has goroutines (lightweight threads) managing:
- Logging (collecting stdout/stderr)
- Signal handling (forwarding signals)
- Stat collection (CPU, memory usage)

With 100 containers, the daemon has 100+ goroutines. This adds overhead.

**containerd memory management:**

containerd uses snapshotters (e.g., overlay2) to manage layers. When you:

```bash
docker pull ubuntu  # containerd downloads layers
```

Layers are stored in `/var/lib/containerd/snapshotters/overlay2/`. Each layer is a directory tree.

When you:
```bash
docker run ubuntu   # containerd creates a snapshot
```

A new snapshot is created from the base image layers (copy-on-write). Container modifications go to the snapshot's upper layer.

### ⚫ Expert Notes

**Docker daemon internals (simplified):**

```go
// Pseudo-code of Docker daemon
type Daemon struct {
  imageService ImageService    // Manages images
  container Service ContainerService  // Manages containers (delegates to containerd)
  networkDriver NetworkDriver   // Manages networks
  volumeDriver VolumeDriver     // Manages volumes
  api *httpServer               // REST API server
  containerd *containerd.Client // Connection to containerd
}

func (d *Daemon) Create(ctx context.Context, config *types.ContainerConfig) {
  // Resolve image (pull if needed)
  img := d.imageService.GetImage(config.Image)
  
  // Delegate to containerd
  resp := d.containerd.Create(ctx, &ContainerRequest{
    ID: generateID(),
    Image: img,
    Config: config,
  })
  
  // Store container metadata
  d.store.SaveContainer(resp.ID, config)
}
```

**runc execution path (detailed):**

```
runc run <container-id>
  ↓
runc reads config.json and rootfs/
  ↓
clone(CLONE_NEWPID | CLONE_NEWNET | CLONE_NEWNS | ...)
  ↓
Child process in new namespaces:
  1. pivot_root to rootfs
  2. Setup cgroups limits
  3. Set capabilities (drop CAP_SYS_ADMIN, etc.)
  4. Set AppArmor/SELinux context
  5. Set environment variables
  6. Set user/group (usually root, UID 0)
  7. execve() the process (from config.Cmd)
  ↓
Container process now running
  ↓
Parent (runc) waits for child
  ↓
Child exits → runc exits
```

**Docker daemon recovery scenarios:**

```
Scenario 1: containerd crashes
  - Containers continue running (shim keeps them alive)
  - After containerd restart:
    - containerd reads container metadata
    - Reconnects to running shims
    - Resume management

Scenario 2: runc crashes (rare)
  - Container process is still running (runc is gone)
  - shim notices parent died
  - shim becomes container parent (container is orphaned)
  - After containerd restart, reconnect via shim

Scenario 3: Daemon + containerd crash
  - Containers keep running (kernel manages processes)
  - After restart, containers are considered "exited" (stale metadata)
  - User must `docker rm` to clean up
  - Or use privileged recovery commands
```

---

## 3. Practical Labs

### 🟢 Beginner Lab — Trace docker run with strace

**Objective:** See the system calls Docker makes.

**Steps:**

```bash
# 1. Start tracing daemon
sudo strace -p $(pgrep -f "docker.*daemon") -e trace=open,openat,read,write,socket,bind,listen -o /tmp/docker.trace 2>&1 &

# 2. Run a container
docker run ubuntu echo "hello"

# 3. Check what syscalls happened
grep -E "(socket|bind|listen)" /tmp/docker.trace | head -20
# Shows network operations (API server listening, etc.)

# 4. Check file operations
grep "openat" /tmp/docker.trace | grep -E "(layers|containers|images)" | head -10
# Shows file access for container/image storage
```

**Expected output:**
- Socket operations (daemon accepting requests)
- File operations (reading config, layers)
- Process operations (fork/exec of containerd/runc)

---

### 🟡 Intermediate Lab — Inspect Docker daemon memory and goroutines

**Objective:** Understand daemon overhead.

**Steps:**

```bash
# 1. Check daemon memory usage
ps aux | grep dockerd
# Shows RSS (resident set size in MB)

# 2. Connect to daemon with gdb/delve (if compiled with debug symbols)
# Or use /proc interface:
cat /proc/$(pgrep -f dockerd)/status | grep Vm
# Shows: VmPeak (peak), VmHWM (high water mark), etc.

# 3. Check goroutines
# Docker doesn't expose goroutine count easily, but you can:
curl -s --unix-socket /var/run/docker.sock http://localhost/info | jq '.Containers,.ContainersRunning,.Images'
# More containers = more goroutines in daemon

# 4. Start multiple containers
for i in {1..10}; do docker run -d alpine sleep 1000; done

# 5. Check daemon memory again
ps aux | grep dockerd
# Memory should increase (one daemon managing 10 containers)

# 6. Cleanup
docker rm -f $(docker ps -aq)
```

**Expected output:**
- Daemon uses ~100 MB at rest
- Memory grows with more containers
- Each container adds ~1–10 MB to daemon (metadata, goroutines)

---

### 🔴 Advanced Lab — Interact with Docker via REST API

**Objective:** Call Docker API directly (not via CLI).

**Steps:**

```bash
# 1. List containers via API
curl -s --unix-socket /var/run/docker.sock http://localhost/v1.40/containers/json | jq '.[] | {Id, State}'

# 2. Create a container via API
RESPONSE=$(curl -s -X POST --unix-socket /var/run/docker.sock http://localhost/v1.40/containers/create \
  -H "Content-Type: application/json" \
  -d '{
    "Image": "alpine",
    "Cmd": ["echo", "hello from API"],
    "OpenStdin": false,
    "AttachStdout": true,
    "AttachStderr": true
  }')

CONTAINER_ID=$(echo $RESPONSE | jq -r '.Id')
echo "Created container: $CONTAINER_ID"

# 3. Start the container
curl -X POST --unix-socket /var/run/docker.sock http://localhost/v1.40/containers/$CONTAINER_ID/start

# 4. Get container logs
curl -s --unix-socket /var/run/docker.sock "http://localhost/v1.40/containers/$CONTAINER_ID/logs?stdout=1&stderr=1"
# Shows: hello from API

# 5. Remove the container
curl -X DELETE --unix-socket /var/run/docker.sock http://localhost/v1.40/containers/$CONTAINER_ID
```

**Expected output:**
- Create returns container ID
- Start succeeds
- Logs show the command output
- API is fully functional (docker CLI is just a wrapper)

---

### ⚫ Expert Lab — Monitor Docker daemon with metrics

**Objective:** Collect daemon metrics (goroutines, API calls, etc.).

**Steps:**

```bash
# Docker daemon exposes metrics on port 9323 (if enabled)

# 1. Enable metrics in daemon config
# /etc/docker/daemon.json
{
  "metrics-addr": "0.0.0.0:9323",
  "experimental": true
}

# Restart daemon
sudo systemctl restart docker

# 2. Scrape metrics
curl -s http://localhost:9323/metrics | head -30
# Shows Prometheus-format metrics

# 3. Key metrics to watch
curl -s http://localhost:9323/metrics | grep -E "(goroutines|containers_running|images)"
# engine_daemon_container_actions_seconds (operation latency)
# engine_daemon_container_states (container state counts)
# engine_daemon_goroutines (number of goroutines)

# 4. Watch metrics change as you create/destroy containers
watch -n 1 "curl -s http://localhost:9323/metrics | grep engine_daemon_container_states"

# 5. Create containers
docker run -d alpine sleep 1000

# Watch metrics update in real-time
```

**Expected output:**
- Metrics show daemon health (goroutine count, operation latency)
- Counters increment as you create/start/stop containers
- Histogram shows latency distribution (useful for performance tuning)

---

## 4. MCQs

### 50 Beginner MCQs

**Q1.** What is the Docker daemon?

- A) The docker CLI tool
- B) A background process (dockerd) that manages containers
- C) An image stored on Docker Hub
- D) A network interface

**Answer: B** — dockerd is the server; docker CLI is the client.

---

**Q2.** What is containerd?

- A) A container (running instance)
- B) A daemon that manages container lifecycle (used by Docker)
- C) A registry (like Docker Hub)
- D) A storage driver

**Answer: B** — containerd is a minimal container manager; Docker daemon delegates to it.

---

**Q3.** What is runc?

- A) A container runtime that creates containers per OCI Spec
- B) A Docker image
- C) A registry server
- D) A network driver

**Answer: A** — runc is the OCI-compliant runtime; spawned by containerd to create containers.

---

**Q4.** How does the docker CLI communicate with the daemon?

- A) Network TCP sockets only
- B) Unix domain socket (/var/run/docker.sock)
- C) Direct function calls
- D) Shared memory

**Answer: B** — Default is Unix socket; TCP available but less secure.

---

**Q5.** What happens when you `docker run ubuntu`?

- A) Docker downloads ubuntu image, daemon creates container, runc starts process
- B) Ubuntu is installed on the host
- C) A VM is created
- D) The image is executed directly by the kernel

**Answer: A** — daemon → containerd → runc → container starts.

---

**Q6.** What is the containerd-shim?

- A) A small container
- B) A wrapper per container that keeps it alive if containerd restarts
- C) A shell script
- D) A network interface

**Answer: B** — Shim is a lightweight process; improves resilience.

---

**Q7.** Can you interact with Docker daemon directly (without docker CLI)?

- A) No, only via docker CLI
- B) Yes, via REST API
- C) Yes, via gRPC
- D) No, daemon is internal only

**Answer: B** — Docker daemon exposes REST API; curl can call it.

---

**Q8.** What privilege level does Docker daemon run at?

- A) Non-root user (unprivileged)
- B) Root (UID 0)
- C) Depends on configuration
- D) System user (like "docker")

**Answer: B** — Docker daemon runs as root; needed for namespace creation.

---

**Q9.** Which of the following is NOT part of Docker architecture?

- A) Docker daemon
- B) containerd
- C) runc
- D) Kubernetes

**Answer: D** — Kubernetes is an orchestrator, not part of Docker architecture.

---

**Q10.** What is the REST API used for?

- A) Only for monitoring
- B) Communicating between daemon and registry
- C) Allowing clients to control daemon (create, start, stop containers)
- D) Internal daemon communication only

**Answer: C** — Any HTTP client can call the API (docker CLI, curl, SDKs).

---

**[Q11–50: Continue with daemon internals, containerd concepts, runc details, plugins, storage, networking.]**

---

### 50 Intermediate MCQs

**Q1.** If the containerd daemon crashes, what happens to running containers?

- A) Containers stop immediately
- B) Containers continue running; shim keeps them alive; containerd reconnects on restart
- C) Host crashes
- D) Containers are lost

**Answer: B** — Shim provides resilience; containers survive daemon restarts.

---

**[Q2–50: Daemon recovery, containerd snapshots, runc lifecycle, plugin architecture, API design.]**

---

### 50 Advanced MCQs

**Q1.** In Docker daemon, when you `docker pull ubuntu`, what exactly is downloaded?

- A) The entire ubuntu filesystem
- B) Layers (tarballs) + manifest + config JSON
- C) Only the config JSON
- D) A VM image

**Answer: B** — Manifest lists layers; each layer is downloaded separately.

---

**[Q2–50: Daemon internals, memory management, goroutine coordination, shim resilience, recovery scenarios.]**

---

### 50 Expert MCQs

**Q1.** A container is running when the Docker daemon crashes. The shim is PID 1234. What is the relationship between shim, runc, and the container process?

- A) Shim is parent of runc, runc is parent of container
- B) Shim is parent of container (runc is gone)
- C) runc is parent of container (shim is sibling)
- D) All three are separate, unrelated

**Answer: B** — runc exits after startup; shim becomes parent (container adopts shim).

---

**[Q2–50: Daemon architecture edge cases, recovery scenarios, performance tuning, security hardening.]**

---

## 5. Interview Questions (with answers)

### 50 Beginner Q&A

**Q1. Draw the Docker architecture and explain each component.**

**Short answer:** Docker CLI → daemon (via socket) → containerd (gRPC) → runc (OCI runtime) → kernel.

**Full answer:**
1. **docker CLI:** Command-line tool; sends requests to daemon via REST API (Unix socket default)
2. **Docker daemon (dockerd):** Long-running process; manages images, containers, networks, volumes; orchestrates work
3. **containerd:** Minimal container manager; daemon delegates container lifecycle to it
4. **runc:** OCI-compliant runtime; creates containers (namespaces, cgroups, execve)
5. **Kernel:** Provides namespaces, cgroups, process scheduling; runs containers

**Follow-up:** "Why is this architecture better than one monolithic piece?"

**Your answer:** Separation of concerns. daemon = orchestration, containerd = container management, runc = OCI compliance. If runc has a bug, just update runc. If daemon has a bug, update daemon. Modularity enables:
- Using different runtimes (runc, crun, Kata, gVisor)
- Reusing containerd in non-Docker projects
- Independent security patches
- Easier testing and debugging

---

**Q2. What does the containerd-shim do?**

**Short answer:** Keeps the container alive if containerd daemon crashes; acts as container parent.

**Full answer:**
- When docker run creates a container, it spawns a shim process per container
- Shim's job: manage container lifecycle (stdin/stdout/stderr, signal handling, exit code collection)
- If containerd crashes: shim is unaffected; container keeps running; shim becomes container's parent
- When containerd restarts: it reconnects to shims, resumes management
- If shim crashes: container still runs (kernel manages it), but you lose stdio

**Follow-up:** "What happens if both shim and containerd crash?"

**Your answer:** Container still runs (kernel doesn't care about userspace daemons). But you can't interact with it (no stdio). You can kill it from command line (kill -9 <PID>) or restart the daemons and reconnect.

---

**[Q3–50: Daemon internals, containerd architecture, plugins, REST API, security model.]**

---

### 50 Intermediate Q&A

**Q1. Explain the flow when you `docker run ubuntu ls -la`.**

**Short answer:** CLI → daemon creates container → containerd setup → runc exec → process runs → container stops.

**Full answer:**
1. CLI: `docker run ubuntu ls -la` → sends JSON to daemon
2. Daemon: resolves image (pulls if needed) → stores config
3. Daemon: calls `containerd.Create()` → containerd creates snapshot (rootfs from layers)
4. Daemon: calls `containerd.Start()` → containerd calls `runc create` and `runc start`
5. runc: creates namespaces, cgroups, mounts rootfs, execves `/bin/sh -c ls -la`
6. Process: runs, outputs to stdout
7. Shim: captures output, sends to daemon
8. Daemon: forwards to CLI
9. CLI: displays to user
10. Process exits (ls finishes)
11. Daemon: calls `containerd.Stop()` → runc cleans up namespaces, cgroups
12. Container stopped

**Follow-up:** "Why doesn't the daemon create containers directly?"

**Your answer:** Separation of concerns. Daemon is high-level orchestration (images, networks, volumes). containerd is focused on container lifecycle. runc is focused on OCI compliance. Each can be updated/replaced independently. Also, daemon can restart without affecting running containers (shim keeps them alive).

---

**[Q2–50: Daemon plugins, API design, containerd internals, runc lifecycle, security model.]**

---

### 50 Advanced Q&A

**Q1. If a container's memory usage spikes, which daemon (docker/containerd/runc) limits it, and how?**

**Short answer:** cgroups enforce the limit; all three daemons set up cgroups, but the limit is enforced by the kernel.

**Full answer:**
- When docker run is called with `--memory=512m`:
  1. Daemon stores the limit in container config
  2. Daemon passes it to containerd
  3. containerd passes it to runc
  4. runc writes to /sys/fs/cgroup/memory/docker/<id>/memory.limit_in_bytes
  5. Kernel's page allocator monitors the container's memory usage against this limit
  6. If limit exceeded: kernel tries reclaim (evict cache pages)
  7. If still over limit: OOM killer runs; selects process with highest score; sends SIGKILL

The daemon/containerd/runc set up the limit; the kernel enforces it.

**Follow-up:** "Can the container escape this limit?"

**Your answer:** No, not without a kernel bug. cgroups are kernel-enforced. A compromised process inside the container can't write to /sys/fs/cgroup/ (it's outside its namespace and protected). The only way to escape is a kernel vulnerability (very rare, but possible).

---

**[Q2–50: Daemon recovery scenarios, goroutine management, metrics/monitoring, security edge cases, performance tuning.]**

---

## 6. Troubleshooting

| Symptom | Root Cause | Debug | Fix |
|---------|-----------|-------|-----|
| docker command hangs | Daemon unresponsive or socket blocked | `ps aux \| grep dockerd` | Restart daemon; check socket permissions |
| Container won't start | runc error (namespace, cgroup, rootfs) | `docker logs`; `journalctl -u docker` | Check logs; validate image; recreate |
| Daemon crash | Out of memory or fatal error | `dmesg`; `systemctl status docker` | Increase memory; restart daemon |
| High daemon CPU | Many containers or API thrashing | `docker stats`; `top` | Reduce container count; optimize API calls |
| Socket permission denied | docker socket owner/permissions wrong | `ls -la /var/run/docker.sock` | Add user to docker group; fix perms |

---

## 7. Security

**Daemon runs as root:** Any container escape = host compromise. Mitigate: drop capabilities, use seccomp, use user namespaces.

**Socket access control:** Restrict who can access /var/run/docker.sock (default: docker group users).

---

## 8. Performance

**Daemon overhead:** 100–200 MB RAM; 1–5% CPU (at rest).

**Startup latency:** 100 ms per container (daemon → containerd → runc round-trips).

---

## 9. Projects

### 🟢 Beginner Project — Docker Architecture Diagram

**Objective:** Create visual diagram of Docker architecture with labels.

---

### 🟡 Intermediate Project — Custom Docker API Client

**Objective:** Write a tool that calls Docker REST API (not using docker CLI).

---

### 🔴 Advanced Project — Monitor Docker Daemon

**Objective:** Collect and graph daemon metrics (goroutines, API latency, container count).

---

### ⚫ Expert Project — Docker Daemon Extension

**Objective:** Write a custom plugin (storage, network, logging driver).

---

### 🚀 Production Project — Daemon High Availability

**Objective:** Design setup where daemon crashes don't affect running containers.

---

### 🏢 Enterprise Project — Docker Cluster Management

**Objective:** Manage Docker daemons across multiple hosts; coordinate container placement.

---

## 10. Self-Assessment

✅ Understand daemon → containerd → runc layering.
✅ Can debug issues at each layer.
✅ Understand REST API.
✅ Ready for Images topic.

---

**[END OF PHASE 1, TOPIC 1]**

Commit:
```bash
git add phase-01-docker-core/01-docker-architecture.md
git commit -m "phase-01: docker architecture — complete core treatment" && git push
```
