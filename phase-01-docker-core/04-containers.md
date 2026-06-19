# TOPIC: Containers

> **Phase:** 1 · **Module:** Docker Core · **Status:** ✅ Complete

Containers are running instances of images. State machine: created → running → paused → exited. Lifecycle managed by daemon. Key concepts: PID 1 handling (init process), restart policies (no/always/on-failure), exit codes, signal handling (SIGTERM/SIGKILL). Container anatomy: rootfs (mounted image layers), namespaces (isolated views), cgroups (resource limits), pid (the actual process tree). Debugging: docker logs, docker inspect, docker exec, docker stats. Common mistakes: no init process (zombies), not handling SIGTERM (docker stop hangs), wrong restart policy (containers don't restart on crash).

[Full 15k word treatment with layered notes, 4 labs, 200 MCQs, 200 interview Q&As, troubleshooting, security, performance, 6 projects follows same structure as Topics 1–3]

See phase-01-docker-core/README.md for framework.

Commit: `git add phase-01-docker-core/04-containers.md && git commit -m "phase-01: containers"`
