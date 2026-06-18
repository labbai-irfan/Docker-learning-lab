# 📖 CURRICULUM — The Complete Docker Knowledge Tree

Every topic in this program. 13 phases · ~90 modules · 300+ topics.
Each topic node is written up using [`_templates/TOPIC_TEMPLATE.md`](_templates/TOPIC_TEMPLATE.md).

---

```
DOCKER UNIVERSITY — MASTER KNOWLEDGE TREE
│
├── PHASE 0 — FOUNDATIONS (Pre-Docker Knowledge)
│   ├── Computing Fundamentals
│   │   ├── Bare Metal vs Virtualized vs Containerized
│   │   ├── CPU / Memory / I/O scheduling
│   │   ├── System Calls (syscalls)
│   │   ├── User Space vs Kernel Space
│   │   └── ABI vs API
│   ├── Virtualization
│   │   ├── Full Virtualization
│   │   ├── Para-virtualization
│   │   ├── Hardware-assisted (VT-x / AMD-V)
│   │   ├── Type 1 Hypervisors (ESXi, Xen, Hyper-V, KVM)
│   │   ├── Type 2 Hypervisors (VirtualBox, VMware Workstation)
│   │   └── VMs vs Containers
│   ├── Linux Internals
│   │   ├── Linux Boot Process
│   │   ├── Processes & PIDs
│   │   ├── Filesystems (ext4, xfs, btrfs)
│   │   ├── Mount & VFS
│   │   ├── Signals
│   │   ├── Capabilities
│   │   ├── chroot / pivot_root
│   │   ├── seccomp
│   │   ├── AppArmor / SELinux
│   │   └── Init systems (systemd, sysvinit, tini)
│   ├── Kernel Primitives
│   │   ├── Namespaces (pid, net, mnt, uts, ipc, user, cgroup, time)
│   │   ├── Control Groups (cgroups v1 vs v2)
│   │   ├── Union Filesystems (OverlayFS, AUFS, devicemapper)
│   │   └── Copy-on-Write (CoW)
│   ├── Networking Fundamentals
│   │   ├── OSI & TCP/IP models
│   │   ├── TCP / UDP / ICMP
│   │   ├── IP addressing / Subnetting / CIDR
│   │   ├── Ports & Sockets
│   │   ├── DNS resolution
│   │   ├── NAT / iptables / nftables
│   │   ├── veth pairs
│   │   └── Linux Bridges
│   └── Container Standards & Ecosystem
│       ├── OCI (Image / Runtime / Distribution specs)
│       ├── CRI (Container Runtime Interface)
│       ├── Runtimes (runc, crun, containerd, CRI-O, gVisor, Kata)
│       ├── Builders (BuildKit, Buildah, Kaniko, img)
│       └── History (chroot → LXC → Docker → containerd → OCI)
│
├── PHASE 1 — DOCKER CORE
│   ├── Docker Architecture (client–server, dockerd, containerd, shim, runc, API/socket, Desktop)
│   ├── Installation & Setup (Linux, Desktop, WSL2, rootless, post-install)
│   ├── Images (anatomy, layers, manifest/config, digests, base images, tags, multi-arch)
│   ├── Containers (lifecycle, state machine, PID 1, restart policies, exit codes, init)
│   ├── Registries (Hub, Harbor/Nexus/Artifactory, ECR/GCR/ACR/GHCR, registry:2, auth, rate limits)
│   └── Storage Drivers (overlay2, fuse-overlayfs, devicemapper, btrfs, zfs, vfs)
│
├── PHASE 2 — DOCKER CLI & COMMANDS
│   ├── Lifecycle: run, create, start, stop, restart, pause, unpause, kill, rm, wait, rename, update
│   ├── Inspect/Debug: ps, logs, inspect, top, stats, events, port, diff, attach, exec, cp
│   ├── Images: build, buildx, image, images, pull, push, tag, rmi, save, load, import, export, commit, history
│   ├── Registry: login, logout, search, scout
│   ├── Networking: network create/ls/inspect/connect/disconnect/rm/prune
│   ├── Volumes: volume create/ls/inspect/rm/prune
│   ├── System: system df/prune/info/events, info, version
│   ├── Orchestration: compose, swarm, service, stack, node, secret, config
│   └── Advanced: plugin, trust, manifest, context, builder, init
│
├── PHASE 3 — DOCKERFILE MASTERY
│   ├── Instructions: FROM, RUN, CMD, ENTRYPOINT, COPY, ADD, ENV, ARG, WORKDIR, EXPOSE,
│   │   LABEL, USER, VOLUME, HEALTHCHECK, SHELL, STOPSIGNAL, ONBUILD, MAINTAINER(dep.)
│   ├── Layer mechanics & build cache
│   ├── Multi-stage builds
│   ├── BuildKit (cache mounts, secrets, SSH, heredocs)
│   ├── .dockerignore
│   ├── ARG vs ENV; CMD vs ENTRYPOINT; exec vs shell form
│   └── Image optimization & reproducible builds
│
├── PHASE 4 — NETWORKING
│   ├── Drivers: bridge, host, none, overlay, macvlan, ipvlan
│   ├── Default vs user-defined bridge
│   ├── Embedded DNS & service discovery
│   ├── Port publishing & EXPOSE
│   ├── Overlay & VXLAN; Swarm ingress/routing mesh
│   ├── CNI overview
│   ├── Load balancing (L4/L7)
│   ├── Reverse proxies: Nginx, Traefik, Caddy, HAProxy
│   ├── Service mesh intro (Envoy, Linkerd)
│   └── Network troubleshooting (nsenter, tcpdump, ss)
│
├── PHASE 5 — STORAGE & DATA
│   ├── Volumes (named/anonymous), Bind mounts, tmpfs
│   ├── Volume drivers & plugins
│   ├── Backends: overlay2, AUFS, ZFS, btrfs
│   ├── Network storage: NFS, CIFS/SMB, EFS, EBS
│   ├── Distributed: Ceph, GlusterFS, Longhorn, Portworx
│   ├── Persistence patterns, Backup/restore
│   └── Stateful workloads
│
├── PHASE 6 — DOCKER COMPOSE
│   ├── Compose spec (v2→v3→unified)
│   ├── Services / networks / volumes / configs / secrets
│   ├── depends_on, healthchecks, conditions
│   ├── Profiles, extends, anchors
│   ├── Env & .env, override files
│   ├── Scaling, Compose Watch
│   └── Compose vs Swarm vs Kubernetes
│
├── PHASE 7 — ORCHESTRATION
│   ├── Swarm (managers/workers, Raft, services/tasks, stacks, overlay, secrets, rolling updates)
│   └── Kubernetes bridge (pods/deploy/svc/ingress, containerd runtime, kubelet/kube-proxy, Helm/Kustomize, Kompose)
│
├── PHASE 8 — SECURITY
│   ├── Threat model & attack surface
│   ├── Image scanning: Trivy, Grype, Docker Scout, Clair, Snyk
│   ├── SBOM (Syft, SPDX, CycloneDX)
│   ├── Signing: DCT/Notary, Cosign/Sigstore
│   ├── Supply chain (SLSA, provenance, attestations)
│   ├── Runtime: Falco, Tracee, AppArmor, SELinux, seccomp
│   ├── Rootless, least privilege, userns remap
│   ├── Secrets management (Vault, BuildKit secrets)
│   └── CIS Benchmark / docker-bench, compliance
│
├── PHASE 9 — OBSERVABILITY & MONITORING
│   ├── Metrics: Prometheus, cAdvisor, node-exporter, dockerd metrics
│   ├── Visualization: Grafana
│   ├── Logging: drivers, Loki, ELK/EFK, Fluentd
│   ├── Tracing: OpenTelemetry, Jaeger, Tempo
│   ├── Alerting: Alertmanager
│   └── Healthchecks, golden signals
│
├── PHASE 10 — CI/CD & AUTOMATION
│   ├── GitHub Actions, GitLab CI, Jenkins
│   ├── ArgoCD / Flux (GitOps)
│   ├── Image build optimization & promotion
│   └── Deployment strategies: rolling, blue-green, canary, A/B, shadow
│
├── PHASE 11 — PRODUCTION ARCHITECTURE
│   ├── Styles: monolith, microservices, event-driven, SOA
│   ├── 12-Factor, statelessness, externalized config
│   ├── HA & redundancy, DR (RTO/RPO)
│   ├── Scaling & autoscaling, resource mgmt
│   └── Multi-env, cost optimization, capacity planning
│
├── PHASE 12 — AI / ML INFRASTRUCTURE
│   ├── GPU containers (NVIDIA toolkit, --gpus), CUDA/cuDNN images
│   ├── LLM serving: Ollama, vLLM, TGI, llama.cpp, SGLang
│   ├── Models: Llama, DeepSeek, Mistral, Qwen, Phi
│   ├── RAG & vector DBs (pgvector, Qdrant, Milvus, Weaviate, Chroma)
│   ├── Frameworks: LangChain, LangGraph, LlamaIndex
│   ├── Agents & MCP servers in containers
│   └── Inference scaling, GPU scheduling (MIG, time-slicing)
│
└── PHASE 13 — CAPSTONE: PORTFOLIO, REPO & CAREER
    ├── Repository structure
    ├── READMEs, progress trackers, journals
    ├── Commit & milestone strategy
    ├── GitHub portfolio strategy
    ├── Certification path (DCA, CKA/CKAD, cloud)
    ├── Interview & job-prep strategy
    └── Role mapping
```

---

## Per-topic deliverables (applied to every node above)

For **each** topic: Definition · Purpose · Internal Working · Architecture ·
Lifecycle · Components · Data Flow · Advantages · Disadvantages · Tradeoffs ·
Production/Industry Usage · Best Practices · Common Mistakes · Security ·
Performance → 🟢🟡🔴⚫ Notes → Labs (4) → MCQs (50×4) → Interview Q&A (50×4) →
Troubleshooting → Security → Performance → Projects (6 levels).
