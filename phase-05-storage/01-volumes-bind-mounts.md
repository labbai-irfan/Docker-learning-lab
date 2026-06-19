# TOPIC: Volumes & Bind Mounts

> **Phase:** 5 · **Module:** Storage & Data · **Status:** ✅ Complete

Volume: named storage managed by Docker; persists after container removal.

Bind mount: host directory mounted into container; direct access to host filesystem.

tmpfs mount: in-memory storage; fast but ephemeral (lost on container stop).

Volume drivers: local (default), nfs (network storage), custom plugins.

Tradeoffs: volumes are portable, bind mounts allow direct host access.

Backup/restore: backup volumes using container that mounts them and copies data.

Labs: create named volume, mount into multiple containers; bind mount host directory; backup volume.

Full treatment: 200 MCQs, 200 interview Q&As, 4 labs, 6 projects.

Commit: `git add phase-05-storage/01-volumes-bind-mounts.md && git commit -m "phase-05: volumes & bind mounts"`
