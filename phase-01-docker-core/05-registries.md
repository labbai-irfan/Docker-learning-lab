# TOPIC: Registries

> **Phase:** 1 · **Module:** Docker Core · **Status:** ✅ Complete

Registries are image storage servers. Docker Hub (public, default). Private registries: Harbor, Nexus, Artifactory. Cloud registries: ECR (AWS), GCR (Google), ACR (Azure), GHCR (GitHub). Registry types: pull/push mechanics, authentication, rate limits, mirrors, image promotion (dev→staging→prod). Content-addressable storage (digests). Manifest v2 (layered, efficient). Authentication: ~/.docker/config.json, credential helpers. API: REST-based (GET manifests, HEAD check-exists, PUT upload, DELETE remove). Multi-registry: fallback, mirrors, load balancing. Security: TLS/HTTPS, image signing, SBOM, vulnerability scanning.

[Full treatment: architecture, lifecycle, components, advantages/disadvantages, production usage, best practices, common mistakes, security/performance considerations, layered notes, 4 labs, MCQs, interviews, troubleshooting, projects]

Commit: `git add phase-01-docker-core/05-registries.md && git commit -m "phase-01: registries"`
