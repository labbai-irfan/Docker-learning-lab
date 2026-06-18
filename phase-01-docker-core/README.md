# Phase 1 — Docker Core

> **Why:** Now that you understand *how* containers work, learn Docker
> specifically — the architecture, images, containers, registries, storage.

---

## Topics

1. **Docker Architecture** — client–server, dockerd, containerd, shim, runc, API/socket, Desktop
2. **Installation & Setup** — Linux, Docker Desktop, WSL2, rootless mode
3. **Images** — anatomy, layers, manifest/config, digests, base images, tags, multi-arch
4. **Containers** — lifecycle, state machine, PID 1, restart policies, exit codes
5. **Registries** — Hub, private (Harbor/Nexus), cloud (ECR/GCR/ACR/GHCR), auth, rate limits
6. **Storage Drivers** — overlay2, fuse-overlayfs, devicemapper, btrfs, zfs

---

## How to use this phase

| Topic | Notes | Labs | MCQs | Q&A | Project |
|-------|:-----:|:----:|:----:|:---:|:-------:|
| Docker Architecture | 🟢🟡🔴⚫ | 4 | 200 | 200 | 6 |
| Installation & Setup | 🟢🟡🔴⚫ | 4 | 200 | 200 | 6 |
| Images | 🟢🟡🔴⚫ | 4 | 200 | 200 | 6 |
| Containers | 🟢🟡🔴⚫ | 4 | 200 | 200 | 6 |
| Registries | 🟢🟡🔴⚫ | 4 | 200 | 200 | 6 |
| Storage Drivers | 🟢🟡🔴⚫ | 4 | 200 | 200 | 6 |

**Estimated time:** 30 hours.

---

## Learning objectives

✅ Can draw the Docker architecture (client, daemon, containerd, shim, runc)  
✅ Can install Docker on Linux, explain post-install steps  
✅ Can explain image layers, digests, manifests  
✅ Can explain container state machine and lifecycle  
✅ Can troubleshoot a container that won't start  
✅ Can configure and use private registries  
✅ Can explain overlay2 driver and when to use others  
✅ Can push/pull images, auth with registries  
✅ Can debug image size bloat  
✅ Comfortable with docker run, docker build, docker inspect  

---

## Key insights

- **Docker uses containerd.** The daemon is becoming a thin orchestration layer;
  containerd handles containers.
- **Images are layers.** Each layer is a tarball; pulling means downloading and extracting.
- **Manifests enable multi-arch.** One image tag can point to different binaries for arm64/amd64.
- **Storage drivers hide complexity.** overlay2 is default; others matter for specific workloads.
- **Registries are dumb storage.** HTTP, content-addressable, stateless. Auth is separate.

---

## Progress

- [ ] Docker Architecture
- [ ] Installation & Setup
- [ ] Images
- [ ] Containers
- [ ] Registries
- [ ] Storage Drivers
