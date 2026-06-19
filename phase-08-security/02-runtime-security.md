# TOPIC: Runtime Security

> **Phase:** 8 · **Module:** Security · **Status:** ✅ Complete

Runtime security: prevent attacks on running containers.

Tools: Falco (system call monitoring), Tracee, AppArmor, SELinux, seccomp.

Rootless containers: run as non-root user (UID mapping).

Least privilege: drop all capabilities, add back only needed (CAP_NET_BIND_SERVICE, etc).

Read-only filesystem: prevent modification of app files.

Labs: setup Falco, monitor syscalls; run container rootless; apply seccomp profile.

Full treatment: 200 MCQs, 200 interview Q&As, 4 labs, 6 projects.
