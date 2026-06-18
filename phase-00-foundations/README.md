# Phase 0 — Foundations

> **Why:** Everything that happens inside a container is powered by Linux kernel
> primitives (namespaces, cgroups, union filesystems). You cannot master Docker
> without understanding these. This is where 70% of hard interview questions come from.

---

## Topics

1. **Computing Fundamentals** — bare metal vs VMs vs containers, syscalls, kernel/user space
2. **Virtualization** — hypervisors, hardware-assisted VM, VMs vs containers tradeoff
3. **Linux Internals** — processes, filesystems, signals, capabilities, chroot, init systems
4. **Kernel Primitives** — namespaces, cgroups, union filesystems, copy-on-write
5. **Networking Fundamentals** — TCP/IP, DNS, NAT, iptables, veth, Linux bridges
6. **Container Standards & Ecosystem** — OCI, CRI, runtimes (runc, containerd), history

---

## How to use this phase

| Topic | Notes | Labs | MCQs | Q&A | Project |
|-------|:-----:|:----:|:----:|:---:|:-------:|
| Computing Fundamentals | 🟢🟡🔴⚫ | 4 | 200 | 200 | 6 |
| Virtualization | 🟢🟡🔴⚫ | 4 | 200 | 200 | 6 |
| Linux Internals | 🟢🟡🔴⚫ | 4 | 200 | 200 | 6 |
| Kernel Primitives | 🟢🟡🔴⚫ | 4 | 200 | 200 | 6 |
| Networking Fundamentals | 🟢🟡🔴⚫ | 4 | 200 | 200 | 6 |
| Container Standards | 🟢🟡🔴⚫ | 4 | 200 | 200 | 6 |

**Estimated time:** 40 hours for full-depth treatment.

---

## Files in this folder

```
phase-00-foundations/
├── README.md (this file)
├── 01-computing-fundamentals.md
├── 02-virtualization.md
├── 03-linux-internals.md
├── 04-kernel-primitives.md
├── 05-networking-fundamentals.md
└── 06-container-standards.md
```

Start with `01-computing-fundamentals.md`. These topics build on each other —
don't skip ahead.

---

## Learning objectives

After Phase 0:

✅ Can explain what a namespace is and why Docker uses them  
✅ Can list the 8 namespace types and draw how they isolate  
✅ Can explain cgroups (v1 and v2) and write a simple cgroup hierarchy  
✅ Can trace a syscall from userland through the kernel  
✅ Can explain copy-on-write and why it matters for image layers  
✅ Can compare containers to VMs (pros/cons) with exact tradeoffs  
✅ Can explain the OCI spec and why it exists  
✅ Can defend why Docker chose runc over other runtimes  
✅ Comfortable with iptables/netfilter basics  
✅ Can write a simple C program that uses clone() with namespace flags  

---

## Key insights

- **Containers are NOT lightweight VMs** — they're groups of processes sharing
  resources via kernel primitives. Very different isolation model.
- **Namespaces are the KEY isolation mechanism.** Every container gets its own
  /proc, network stack, mounts, pid space, etc. by namespace.
- **cgroups control *resources*.** Even if a process escapes its namespace, it
  stays throttled by cgroup limits.
- **Union filesystems make images efficient.** Layers reuse filesystem blocks
  via CoW. Crucial for understanding image size, build performance, and storage drivers.
- **OCI standardized containers.** Docker uses OCI Image Spec and Runtime Spec,
  which means runc, crun, containerd all speak the same language.

---

## Common mistakes

❌ "Containers are VMs with less overhead" — wrong mental model  
❌ "Namespaces provide security" — they don't; seccomp/AppArmor do  
❌ "All container runtimes are the same" — they have different security/perf tradeoffs  
❌ "cgroups v1 is deprecated" — still widely used, but v2 is the future  
❌ "I don't need to know kernel stuff if I just use Docker" — you do; debugging requires it  

---

## Progress

- [ ] Computing Fundamentals — 40 hrs to read/labs/MCQs
- [ ] Virtualization — 40 hrs
- [ ] Linux Internals — 40 hrs
- [ ] Kernel Primitives — 40 hrs (hardest)
- [ ] Networking Fundamentals — 30 hrs
- [ ] Container Standards — 20 hrs

**Total: ~40 hours actual learning** (if you don't redo things).
