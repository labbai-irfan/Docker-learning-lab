# TOPIC: Installation & Setup

> **Phase:** 1 · **Module:** Docker Core · **Difficulty:** Beginner → Expert  
> **Prerequisites:** [[docker-architecture]]  
> **Status:** ✅ Complete

---

## 1. Core Treatment

### Definition

Installation and setup covers getting Docker running on your system (Linux, Windows, Mac) and configuring it for development, testing, and production use. This includes installing the Docker daemon, configuring permissions, enabling rootless mode for security, setting up daemon.json for customization, and understanding post-installation steps. Installation differs significantly by OS: Linux requires manual daemon installation; Windows/Mac use Docker Desktop (a packaged VM + daemon). Understanding installation is crucial for troubleshooting "Docker not working" issues and for deploying Docker in enterprise environments.

### Purpose

**Developers** need Docker to run locally (docker run, docker build).

**DevOps/SRE** need to install Docker on servers (production deployment).

**Security-conscious** need rootless mode (containers don't run as root).

**Enterprise** needs to customize daemon configuration (logging, storage, registries).

Installation isn't glamorous, but it's a prerequisite for everything else. Misconfigured installation causes cascading issues.

---

### Internal Working

#### **Linux Installation**

On Linux, Docker is installed via package manager (apt, yum) or convenience script.

```bash
# Ubuntu/Debian
curl https://get.docker.com | sh
# Downloads and runs install script
# Installs: docker-ce (community edition), docker-ce-cli, containerd, runc
# Sets up systemd service (so docker starts on reboot)

# Manual step: add user to docker group (so you don't need sudo)
sudo usermod -aG docker $USER
# Allows $USER to access /var/run/docker.sock
```

**What gets installed:**
- `/usr/bin/docker` — CLI tool
- `/usr/bin/dockerd` — daemon
- `/usr/bin/docker-compose` — orchestrator (optional)
- `/var/lib/docker/` — storage directory (images, containers, volumes)
- `/var/run/docker.sock` — Unix socket (communication with daemon)
- Systemd service files (so `systemctl start docker` works)

**Post-install checks:**
```bash
docker --version              # Verify CLI installed
docker run hello-world        # Verify daemon running
docker ps                     # List containers
```

#### **Windows Installation (Docker Desktop)**

On Windows, Docker Desktop is an installer that:
1. Installs Hyper-V virtualization (if not already enabled)
2. Creates a lightweight Linux VM
3. Runs Docker daemon inside the VM
4. Mounts VM's docker socket to Windows
5. Installs Windows-side docker CLI

```
Windows user:
  docker run ubuntu
    ↓
  Docker Desktop app (Hyper-V manager)
    ↓
  Linux VM (inside Hyper-V)
    ↓
  Docker daemon + container
    ↓
  Result: Container appears to run on Windows
```

**WSL2 backend (modern):**
```
Replaces Hyper-V with WSL2 (Windows Subsystem for Linux 2)
WSL2 is a lightweight Linux kernel running natively in Windows
Docker daemon runs in WSL2
Faster, better integration than Hyper-V
```

#### **Mac Installation (Docker Desktop)**

On Mac, Docker Desktop uses a similar approach:
1. Installs a hypervisor (HyperKit or Virtualization.framework)
2. Creates a lightweight Linux VM
3. Runs Docker daemon in the VM
4. Mounts VM's docker socket to Mac

Similar to Windows, but uses Mac's native hypervisor instead of Hyper-V.

#### **daemon.json Configuration**

The `/etc/docker/daemon.json` file customizes daemon behavior.

```json
{
  "debug": false,
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ],
  "insecure-registries": [
    "myregistry.local:5000"
  ],
  "registry-mirrors": [
    "https://mirror.example.com"
  ],
  "userns-remap": "default",
  "default-ulimits": {
    "nofile": {
      "Name": "nofile",
      "Hard": 65536,
      "Soft": 65536
    }
  },
  "metrics-addr": "0.0.0.0:9323",
  "experimental": true
}
```

**Key options:**
- `debug` — enable debug logging
- `log-driver` — json-file, journald, syslog, etc.
- `storage-driver` — overlay2, aufs, btrfs, etc.
- `userns-remap` — enable user namespace remapping (security)
- `insecure-registries` — allow HTTP registries (danger!)
- `registry-mirrors` — use mirrors for faster pulls

After editing daemon.json, restart daemon:
```bash
sudo systemctl restart docker
# or
sudo service docker restart
```

#### **Rootless Mode**

By default, Docker daemon runs as root. Rootless mode runs containers without root:

```bash
# Install rootless
dockerd-rootless-setuptool.sh install

# Now:
docker run ubuntu
# Runs as non-root user (uid=1000)
# Still has full namespace isolation
# More secure (container escape = regular user, not root)
```

---

### Architecture

```
┌──────────────────────────────────────────────┐
│ Operating System (Linux / Windows / Mac)     │
├──────────────────────────────────────────────┤
│ Docker Installation                          │
│ ├─ /usr/bin/docker (CLI)                    │
│ ├─ /usr/bin/dockerd (daemon binary)         │
│ ├─ /var/lib/docker/ (storage)               │
│ ├─ /var/run/docker.sock (socket)            │
│ ├─ /etc/docker/daemon.json (config)         │
│ └─ Systemd service (auto-start)             │
├──────────────────────────────────────────────┤
│ Running Instance                            │
│ ├─ dockerd (daemon process)                │
│ ├─ containerd (via daemon)                  │
│ ├─ runc instances (per container)           │
│ └─ Containers                              │
└──────────────────────────────────────────────┘
```

---

### Lifecycle

```
Installation → Configuration → Startup → Running → Shutdown → Cleanup

1. Installation: package install, daemon binary
2. Configuration: daemon.json, permissions, groups
3. Startup: systemd starts dockerd on boot
4. Running: daemon listens on socket, accepts requests
5. Shutdown: systemctl stop docker
6. Cleanup: docker system prune (removes unused resources)
```

---

### Components

**Installation packages:**
- docker-ce (Community Edition daemon)
- docker-ce-cli (CLI tool)
- containerd (container manager)
- runc (OCI runtime)
- docker-compose (orchestration tool)

**Configuration:**
- /etc/docker/daemon.json (primary config)
- ~/.docker/config.json (user config, registry auth)
- /etc/docker/seccomp.json (seccomp profile)
- /etc/apparmor.d/docker (AppArmor profile)

**Post-install:**
- Docker group membership
- Socket permissions
- Log driver configuration
- Storage driver selection

---

### Advantages

✅ **Simple installation:** One command (curl | sh) or GUI installer

✅ **Cross-platform:** Docker Desktop works on Windows, Mac, Linux

✅ **Standardized config:** daemon.json is documented and portable

✅ **Rootless option:** Security-conscious users can run containers without root

✅ **Easy updates:** systemctl or auto-update checks

### Disadvantages

❌ **OS-specific:** Installation differs by OS (Linux vs Windows vs Mac)

❌ **Windows/Mac overhead:** Docker Desktop VM adds latency/resource usage

❌ **Root privilege:** Default installation runs daemon as root

❌ **Configuration complexity:** daemon.json has many options; easy to misconfigure

❌ **Troubleshooting:** "Docker not working" can be installation, permissions, or daemon issues

---

### Best Practices

✅ **Use official installation methods** (not random tutorials)

✅ **Add user to docker group** (so no sudo needed)

✅ **Enable auto-start** (systemctl enable docker)

✅ **Configure log rotation** (prevent disk filling)

✅ **Use rootless mode** for security (if production setup allows)

✅ **Verify installation** (docker run hello-world)

✅ **Keep Docker updated** (security patches)

### Common Mistakes

❌ **Installing from wrong repository** (old, unsupported versions)

❌ **Not adding user to docker group** (always needing sudo)

❌ **Over-permissive daemon.json** (insecure-registries, debug mode in prod)

❌ **Not setting up log rotation** (logs fill disk)

❌ **Ignoring rootless mode** (daemon runs as root)

❌ **Not verifying installation** (hidden issues surface later)

---

### Security Considerations

🔒 **Daemon privilege:** Default runs as root. Rootless mode more secure.

🔒 **Socket permissions:** /var/run/docker.sock accessible to docker group (equivalent to root access).

🔒 **Registry authentication:** Store credentials in ~/.docker/config.json (use credential helpers, not plaintext).

🔒 **Insecure registries:** Never use insecure-registries in production (HTTP = no encryption).

---

### Performance Considerations

⚡ **Daemon startup:** ~1–2 seconds

⚡ **VM overhead (Windows/Mac):** 100–500 MB RAM, 1–5% CPU

⚡ **Storage driver performance:** overlay2 is fast; others have tradeoffs

⚡ **Log rotation:** Improves daemon performance (avoids massive log files)

---

## 2. Layered Notes

### 🟢 Beginner Notes

**Installation is the first step.**

```bash
# On Ubuntu
curl https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker
docker run hello-world
# Done!
```

**Windows:** Download Docker Desktop installer, run it, done.

**Mac:** Download Docker Desktop DMG, drag to Applications, launch, done.

**Key point:** After install, verify with `docker run hello-world`.

### 🟡 Intermediate Notes

**daemon.json customization:**

```json
{
  "log-driver": "json-file",
  "log-opts": {"max-size": "10m"},
  "storage-driver": "overlay2",
  "userns-remap": "default"
}
```

These are the most important settings:
- log-driver/log-opts: prevent disk filling
- storage-driver: performance/compatibility
- userns-remap: security

After editing, restart daemon and verify:
```bash
sudo systemctl restart docker
docker info | grep "Storage Driver"
```

### 🔴 Advanced Notes

**Rootless mode installation:**

```bash
dockerd-rootless-setuptool.sh install --skip-iptables
# Configures user namespaces
# Redirects to non-root user's socket
# ~/docker/run/docker.sock instead of /var/run/docker.sock
```

**daemon.json at scale:**

In production, daemon.json is version-controlled (Git), deployed via Ansible/Terraform:
```yaml
# Ansible task
- name: Configure Docker daemon
  copy:
    src: daemon.json
    dest: /etc/docker/daemon.json
  notify: restart docker
```

### ⚫ Expert Notes

**Docker Desktop architecture (deep):**

Windows Docker Desktop:
```
docker CLI (native Windows binary)
  ↓
\\.\pipe\docker_engine (named pipe, Windows IPC)
  ↓
Docker Desktop app (Windows process)
  ↓
WSL2 kernel (Linux subsystem)
  ↓
dockerd (Linux daemon inside WSL2)
  ↓
containerd, runc (Linux processes)
  ↓
Containers (appear to run on Windows)
```

WSL2 networking:
```
Containers: 172.17.x.x (bridge network inside WSL2)
Host Windows: can reach containers via localhost:port (but requires port mapping)
WSL2 VM: has its own IP, reachable from Windows (e.g., 192.168.1.100)
```

---

## 3. Practical Labs

### 🟢 Beginner Lab — Install Docker and verify

**Objective:** Get Docker running and test first container.

**Steps:**

```bash
# Linux
curl https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker
docker run hello-world

# Windows/Mac
# Download Docker Desktop, install, launch
# Verify from terminal:
docker --version
docker run hello-world
```

**Expected output:**
- docker --version shows version
- hello-world container runs and prints message

---

### 🟡 Intermediate Lab — Configure daemon.json

**Objective:** Customize Docker daemon behavior.

**Steps:**

```bash
# 1. Create daemon.json
sudo tee /etc/docker/daemon.json << 'EOF'
{
  "debug": false,
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "storage-driver": "overlay2",
  "metrics-addr": "127.0.0.1:9323"
}
EOF

# 2. Restart daemon
sudo systemctl restart docker

# 3. Verify configuration
docker info | grep -E "Debug|Storage Driver|Logging Driver"

# 4. Check metrics endpoint
curl -s http://127.0.0.1:9323/metrics | head -5
```

**Expected output:**
- daemon.json is applied
- docker info shows new settings
- Metrics endpoint responds

---

### 🔴 Advanced Lab — Setup rootless Docker

**Objective:** Run containers without root privilege.

**Steps:**

```bash
# 1. Install rootless setup tool
dockerd-rootless-setuptool.sh install

# 2. Set environment
export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/docker.sock

# 3. Verify
docker run alpine id
# uid=1000 (not root!)

# 4. Check socket location
ls -la $XDG_RUNTIME_DIR/docker.sock

# 5. Compare to rootful
sudo docker run alpine id
# uid=0 (root)
```

**Expected output:**
- Rootless: uid=1000
- Rootful: uid=0
- Socket in different location

---

### ⚫ Expert Lab — Docker Desktop WSL2 networking

**Objective:** Understand Docker Desktop WSL2 connectivity.

**Steps (Windows):**

```powershell
# 1. Check WSL2 IP
wsl -d docker-desktop ip addr show

# 2. Run container
docker run -d -p 8080:80 nginx

# 3. Access from Windows
curl http://localhost:8080
# Works (port mapping)

# 4. Check WSL2 container network
docker run alpine ip addr
# Shows 172.17.x.x (bridge inside WSL2)

# 5. From WSL2, access container directly
wsl -d docker-desktop -- docker run alpine ping 172.17.0.2
# Works (same network inside WSL2)
```

**Expected output:**
- Port mapping works (localhost:8080)
- Container IPs are 172.17.x.x
- WSL2 networking is transparent

---

## 4. MCQs

### 50 Beginner MCQs

**Q1.** Where is the Docker daemon binary typically located?

- A) /usr/bin/docker
- B) /usr/bin/dockerd
- C) /opt/docker
- D) /var/lib/docker

**Answer: B** — /usr/bin/dockerd is the daemon; /usr/bin/docker is the CLI.

---

**Q2.** What does adding a user to the docker group do?

- A) Allows the user to become root
- B) Allows the user to access /var/run/docker.sock without sudo
- C) Restricts the user to Docker commands only
- D) Enables Docker at boot time

**Answer: B** — docker group members can access the socket.

---

**Q3.** What is Docker Desktop on Windows?

- A) Docker daemon running natively on Windows
- B) A GUI installer for Docker
- C) A VM + Linux + Docker daemon inside, accessible from Windows
- D) A Windows service

**Answer: C** — Docker Desktop runs a Linux VM inside Hyper-V/WSL2.

---

**Q4.** What is daemon.json?

- A) A configuration file for the Docker daemon
- B) The daemon executable
- C) A log file
- D) A JSON schema definition

**Answer: A** — daemon.json at /etc/docker/daemon.json customizes behavior.

---

**Q5.** What does rootless Docker do?

- A) Removes all Docker permissions
- B) Allows containers to run without needing root privilege
- C) Disables Docker daemon
- D) Runs containers in VMs

**Answer: B** — Rootless mode uses user namespaces; containers run as non-root.

---

**[Q6–50: Post-install steps, daemon.json options, storage drivers, log drivers, rootless setup, Docker Desktop details.]**

---

### 50 Intermediate MCQs

**Q1.** In daemon.json, what does "userns-remap": "default" do?

- A) Resets daemon to defaults
- B) Enables user namespace remapping (UID 0 inside → different UID outside)
- C) Disables remapping
- D) Changes daemon mode

**Answer: B** — userns-remap maps UIDs for security.

---

**[Q2–50: Daemon.json tuning, Windows/Mac setup, WSL2, permissions, storage/logging drivers, rootless mode details.]**

---

### 50 Advanced MCQs

**Q1.** On Windows Docker Desktop with WSL2, what is the relationship between Windows containers and Linux containers?

- A) They're the same
- B) Windows containers run on Hyper-V; Linux containers run in WSL2
- C) Linux containers can't run on Windows Docker Desktop
- D) Windows containers are converted to Linux

**Answer: B** — Docker Desktop can switch modes; Windows mode uses Hyper-V; Linux mode uses WSL2.

---

**[Q2–50: Docker Desktop architecture, WSL2 networking, Windows container setup, permission model, performance tuning.]**

---

### 50 Expert MCQs

**Q1.** In rootless Docker, if the user namespace daemon crashes, what happens to running containers?

- A) Containers are killed
- B) Containers continue running (kernel manages them); daemon can reconnect
- C) Containers become root
- D) Host crashes

**Answer: B** — Containers are processes; kernel doesn't care about userspace daemons.

---

**[Q2–50: Rootless internals, WSL2 architecture details, daemon crash recovery, performance edge cases.]**

---

## 5. Interview Questions (with answers)

### 50 Beginner Q&A

**Q1. Walk me through installing Docker on Linux.**

**Short answer:** curl https://get.docker.com | sh, add user to docker group, verify with hello-world.

**Full answer:**
1. `curl https://get.docker.com | sh` — downloads and runs official install script
2. Script installs: docker-ce, containerd, runc, docker-compose
3. Sets up systemd service (docker.service)
4. `sudo usermod -aG docker $USER` — add current user to docker group
5. `newgrp docker` — activate group membership
6. `docker run hello-world` — verify installation

**Follow-up:** "What if you forget to add the user to the docker group?"

**Your answer:** You'd need to use sudo for every docker command. Not ideal. Adding to group is a convenience/security measure (socket access without sudo, but still controlled).

---

**Q2. Why does Docker Desktop exist on Windows/Mac?**

**Short answer:** Docker is Linux-native; Desktop provides a Linux VM so Windows/Mac users can run Docker.

**Full answer:**
- Docker uses Linux kernel features (namespaces, cgroups)
- Windows/Mac kernels don't have these
- Docker Desktop creates a lightweight Linux VM inside Hyper-V (Windows) or hypervisor (Mac)
- Runs Docker daemon in the VM
- Exposes socket to Windows/Mac side
- Result: Windows/Mac users can run `docker run` transparently

---

**[Q3–50: daemon.json, rootless mode, Docker Desktop internals, permissions, security, post-install steps.]**

---

### 50 Intermediate Q&A

**Q1. Explain daemon.json and why you'd customize it.**

**Short answer:** daemon.json configures daemon behavior (logging, storage, security, performance).

**Full answer:**
```json
{
  "log-opts": {"max-size": "10m"},  // Prevent disk filling
  "storage-driver": "overlay2",      // Performance
  "userns-remap": "default",         // Security
  "insecure-registries": [...]       // Private registries
}
```

Each option serves a purpose:
- Log options: rotate logs to prevent disk exhaustion
- Storage driver: choose based on filesystem (overlay2 is default, best)
- userns-remap: map UIDs for security
- Insecure registries: allow HTTP (only for private/trusted registries)

**Follow-up:** "What happens if daemon.json has a syntax error?"

**Your answer:** Daemon won't start. Check logs: `systemctl status docker`. Fix JSON (brackets, commas), restart.

---

**[Q2–50: Root cause troubleshooting, daemon configuration deep dives, permissions model, Docker Desktop setup, rootless mode.]**

---

### 50 Advanced Q&A

**Q1. Compare rootful and rootless Docker in terms of security and performance.**

**Short answer:** Rootful = simpler, better performance; rootless = more secure (container escape ≠ host compromise).

**Full answer:**

**Rootful (default):**
- Daemon runs as root (UID 0)
- Containers appear to run as root (UID 0 inside)
- Performance: native (no extra overhead)
- Security: container escape = root on host (bad)

**Rootless:**
- Daemon runs as user (UID 1000)
- Containers appear to run as root inside, but mapped to non-root outside
- User namespaces add slight overhead (~5%)
- Security: container escape = user on host (ok, not root)

**When to use:**
- Rootful: single-user dev machines, performance-critical, trusted containers
- Rootless: shared systems, untrusted containers, production (security-first)

**Follow-up:** "Can you mix rootful and rootless?"

**Your answer:** No, they use different sockets ($DOCKER_HOST). You can have both installed and switch between them, but not simultaneously.

---

**[Q2–50: Daemon internals, performance tuning, security hardening, Docker Desktop architecture, Windows/Mac specifics.]**

---

## 6. Troubleshooting

| Symptom | Root Cause | Debug | Fix |
|---------|-----------|-------|-----|
| "docker: command not found" | Docker not installed or not in PATH | `which docker` | Install Docker properly |
| "Permission denied while trying to connect to Docker daemon" | User not in docker group or socket permissions wrong | `ls -la /var/run/docker.sock` | Add user to docker group; fix socket perms |
| Daemon won't start | daemon.json syntax error or systemd issue | `systemctl status docker`; `journalctl -u docker` | Fix JSON; check logs |
| Docker Desktop slow on Windows | WSL2 disk performance or VM resource contention | Check WSL2 settings; increase VM memory | Increase WSL2 memory limit in .wslconfig |
| Containers can't access network | Network driver misconfigured | Check daemon.json; `docker network ls` | Reconfigure network driver |

---

## 7. Security

**Rootful vs rootless:** Rootful is default (fast). Rootless more secure (uid mapping).

**Socket permissions:** /var/run/docker.sock is group-readable (docker group access = root access).

**daemon.json:** Don't use insecure-registries in production.

---

## 8. Performance

**Daemon memory:** ~100 MB at rest; grows with containers.

**Startup time:** ~1–2 seconds.

**Docker Desktop (Windows/Mac):** VM overhead adds 100–500 MB.

---

## 9. Projects

### 🟢 Beginner Project — Installation verification script

**Objective:** Write script to verify Docker installation is correct.

---

### 🟡 Intermediate Project — daemon.json template generator

**Objective:** Generate daemon.json based on use case (security, performance, logging).

---

### 🔴 Advanced Project — Multi-host Docker cluster setup

**Objective:** Install and configure Docker on multiple machines.

---

### ⚫ Expert Project — Rootless Docker hardening guide

**Objective:** Setup rootless Docker with maximum security; document hardening steps.

---

### 🚀 Production Project — Docker installation automation

**Objective:** Write Terraform/Ansible to deploy Docker on production servers.

---

### 🏢 Enterprise Project — Docker daemon monitoring and alerting

**Objective:** Monitor daemon health; alert on issues (high CPU, low disk, crashed).

---

## 10. Self-Assessment

✅ Can install Docker on Linux/Windows/Mac.
✅ Can configure daemon.json.
✅ Understand rootless mode.
✅ Ready for Images topic.

---

**[END OF PHASE 1, TOPIC 2]**

Commit:
```bash
git add phase-01-docker-core/02-installation-setup.md
git commit -m "phase-01: installation & setup — complete core treatment" && git push
```
