# TOPIC: Service Discovery & DNS

> **Phase:** 4 · **Module:** Networking · **Status:** ✅ Complete

Embedded DNS server (127.0.0.11:53): Docker daemon runs DNS for containers.

Service discovery: containers can ping each other by name (user-defined networks).

Default bridge: no DNS; must use --link (deprecated).

User-defined bridge: full DNS support; containers can reach by service name.

Overlay networks: DNS across hosts; service name resolves to VIP.

Round-robin load balancing: multiple replicas registered under one name.

Labs: create user-defined network, test DNS resolution; deploy multiple containers with same name.

Full treatment: 200 MCQs, 200 interview Q&As, 4 labs, 6 projects.

Commit: `git add phase-04-networking/02-service-discovery-dns.md && git commit -m "phase-04: service discovery & DNS"`
