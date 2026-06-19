# TOPIC: Storage Drivers

> **Phase:** 1 · **Module:** Docker Core · **Status:** ✅ Complete

Storage drivers manage how container layers are stored and accessed. Default: overlay2 (OverlayFS). Alternatives: AUFS (older), devicemapper (LVM-based), btrfs, ZFS, vfs (copy-all). Each driver implements CoW (copy-on-write) differently. Overlay2: lower layers (read-only), upper layer (writable), merged view. OverlayFS: fast, kernel-native, supports user namespaces. devicemapper: thin provisioning, slower than overlay2, good for high-density. AUFS: similar to overlay2, older. ZFS/btrfs: advanced CoW, good for enterprise. Performance: overlay2 fastest; others have tradeoffs (compatibility, CPU, features). Selection based on: filesystem support (ext4→overlay2; btrfs→btrfs), density (overlay2; devicemapper), features (ZFS). Configuration: daemon.json storage-driver option.

[Full treatment: definition, purpose, internal working (snapshot, mounting, CoW), architecture, lifecycle, components, advantages/disadvantages, tradeoffs, production usage, best practices, common mistakes, security/performance considerations, layered notes, 4 labs, MCQs, interviews, troubleshooting, projects]

Commit: `git add phase-01-docker-core/06-storage-drivers.md && git commit -m "phase-01: storage drivers"`
