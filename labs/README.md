# 🧪 Labs Directory

Runnable, hands-on laboratory code for every topic.

---

## Structure

```
labs/
├── phase-00-foundations/
│   ├── 01-namespaces/
│   │   ├── ns-pid-demo.sh          # Demonstrate PID namespace
│   │   ├── ns-net-demo.sh          # Network namespace
│   │   └── README.md               # Run steps, expected output
│   ├── 02-cgroups/
│   │   ├── cgroup-limit.sh
│   │   └── README.md
│   └── ...
├── phase-01-docker-core/
│   ├── 01-architecture/
│   │   ├── inspect-daemon.sh       # Inspect Docker architecture
│   │   └── README.md
│   ├── 02-images/
│   │   ├── build-scratch.Dockerfile
│   │   ├── multi-stage.Dockerfile
│   │   └── README.md
│   └── ...
├── phase-02-cli-commands/
│   ├── lifecycle.sh                # Demo all container lifecycle commands
│   ├── inspect-debug.sh            # Demo inspection commands
│   └── README.md
├── phase-03-dockerfile/
│   ├── optimization/
│   │   ├── Dockerfile.bloated      # 500 MB image
│   │   ├── Dockerfile.optimized    # 50 MB image
│   │   └── README.md               # Layer-by-layer comparison
│   ├── multi-stage/
│   └── buildkit/
├── phase-04-networking/
│   ├── bridge-demo.sh
│   ├── overlay-swarm.sh
│   └── dns-service-discovery.sh
├── ...
├── phase-12-ai-infrastructure/
│   ├── ollama-docker/              # Dockerfile for Ollama
│   ├── vllm-inference/             # vLLM serving container
│   ├── rag-compose/                # Multi-container RAG stack
│   └── gpu-demo.sh                 # Check GPU access
└── README.md (this file)
```

---

## Running labs

Each lab has a README explaining:

1. **Objective** — what you'll learn
2. **Prerequisites** — what you need installed
3. **Steps** — commands to run
4. **Expected output** — what success looks like
5. **Cleanup** — how to remove containers/volumes/networks

Example:

```bash
cd labs/phase-01-docker-core/02-images/
bash build-scratch.Dockerfile   # or docker build ...
```

---

## Lab types

- **Shell scripts** (`.sh`) — run bash directly
- **Dockerfiles** — `docker build`
- **Compose files** (`docker-compose.yml`) — `docker compose up`
- **READMEs** — step-by-step instructions

---

## Total labs

- Phase 0: ~24 labs (4 topics × 4 levels + foundations)
- Phase 1–12: ~20 labs per phase
- **Total: ~250+ runnable labs**

Each is designed to be completed in 15–60 minutes.

---

## Tips

- **Don't skip labs.** They lock in understanding.
- **Modify & experiment.** "What if I change this flag?"
- **Troubleshoot.** If it fails, use docker logs/inspect to debug.
- **Document your findings.** Note what surprised you in JOURNAL.md.
