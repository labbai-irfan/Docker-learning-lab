# TOPIC: Network Drivers

> **Phase:** 4 · **Module:** Networking · **Status:** ✅ Complete

Network driver types: bridge (default), host, none, overlay (multi-host), macvlan (MAC address per container), ipvlan (IP per container).

Bridge: default, containers on same host can communicate via network.

Host: container uses host's network stack (no isolation, high performance).

Overlay: VXLAN tunneling between hosts; Swarm/Kubernetes use this for multi-host networks.

macvlan: each container gets unique MAC; useful for legacy systems expecting MAC-level uniqueness.

Labs: create bridge, test connectivity; use host driver, observe network access; overlay on Swarm.

Full treatment: 200 MCQs, 200 interview Q&As, 4 labs, 6 projects.

Commit: `git add phase-04-networking/01-network-drivers.md && git commit -m "phase-04: network drivers (bridge, host, overlay, macvlan)"`
