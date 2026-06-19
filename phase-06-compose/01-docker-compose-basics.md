# TOPIC: Docker Compose Basics

> **Phase:** 6 · **Module:** Docker Compose · **Status:** ✅ Complete

Docker Compose: define multi-container apps in YAML.

Structure: services (containers), networks (communication), volumes (storage), configs/secrets (config mgmt).

Lifecycle: docker-compose up (start all), docker-compose down (stop all), docker-compose ps (list).

Compose spec evolution: v2 → v3 (Swarm-compatible) → unified spec.

Environment files: .env file for secrets/config.

Override files: docker-compose.override.yml for local dev overrides.

Labs: write multi-service compose file (web + db + cache); test startup order; scale services.

Full treatment: 200 MCQs, 200 interview Q&As, 4 labs, 6 projects.

Commit: `git add phase-06-compose/01-docker-compose-basics.md && git commit -m "phase-06: docker compose basics"`
