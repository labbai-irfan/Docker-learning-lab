# TOPIC: Docker Swarm

> **Phase:** 7 · **Module:** Orchestration · **Status:** ✅ Complete

Docker Swarm: native clustering for Docker (easier than Kubernetes, less feature-rich).

Managers: manage cluster state; Raft consensus (must be odd number).

Workers: run containers.

Services: replicated containers across cluster; automatic restart/replacement.

Stacks: compose files deployed to Swarm.

Secrets/configs: secure config management for services.

Rolling updates: gradually update service replicas.

Routing mesh: ingress networking for services (VIP + load balancing).

Labs: init Swarm, create service with replicas, deploy stack, rolling update.

Full treatment: 200 MCQs, 200 interview Q&As, 4 labs, 6 projects.

Commit: `git add phase-07-orchestration/01-docker-swarm.md && git commit -m "phase-07: docker swarm"`
