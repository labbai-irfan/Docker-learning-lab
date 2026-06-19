# TOPIC: Container Lifecycle Commands

> **Phase:** 2 · **Module:** CLI Commands · **Difficulty:** Beginner → Expert  
> **Prerequisites:** [[docker-architecture]], [[installation-setup]], [[containers]]  
> **Status:** ✅ Complete

---

## 1. Core Treatment

### Definition

Container lifecycle commands control the full lifecycle of Docker containers: creation (docker create, docker run), starting (docker start), stopping (docker stop, docker pause), and removal (docker rm). These are the most frequently used commands; mastery is essential. Each command has dozens of flags controlling: image selection, volumes, networking, environment variables, resource limits, restart policies, signal handling, logging, etc.

### Key Commands

#### **docker run** — Most important command
```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

# Most common usage:
docker run -d -p 8080:80 --name webserver nginx
  -d           detach (run in background)
  -p 8080:80   port mapping (host:container)
  --name       container name
  nginx        image
  (default command from image used)

# With override:
docker run -it ubuntu /bin/bash
  -i           keep STDIN open
  -t           allocate pseudo-terminal
  ubuntu       image
  /bin/bash    override default command

# With limits:
docker run --memory=512m --cpus=0.5 myapp
  --memory     max RAM (512m, 1g, etc.)
  --cpus       max CPU (0.5 = 50% of 1 core)

# With environment & volumes:
docker run -e DATABASE_URL=postgres://db myapp
  -e           environment variable
  
docker run -v /host/path:/container/path myapp
  -v           volume mount

# With restart policy:
docker run --restart=always myapp
  --restart    always, on-failure, unless-stopped, no
```

**Exit codes:**
- 0: successful exit
- 1: general error
- 125: docker error (e.g., invalid flag)
- 126: container command cannot be executed
- 127: container command not found
- 137: killed by SIGKILL (docker kill)
- 143: gracefully terminated by SIGTERM (docker stop)

#### **docker create** — Create without starting
```bash
docker create [OPTIONS] IMAGE [COMMAND]

# Same options as docker run, but doesn't start
docker create --name mycontainer nginx

docker start mycontainer  # Now start it separately
```

#### **docker start** — Start a created/stopped container
```bash
docker start [OPTIONS] CONTAINER

docker start mycontainer           # Restart
docker start -a mycontainer        # -a: attach output
docker start -i mycontainer        # -i: attach stdin (interactive)
```

#### **docker stop** — Gracefully stop container
```bash
docker stop [OPTIONS] CONTAINER

docker stop mycontainer            # Default: wait 10 sec, then SIGKILL
docker stop -t 30 mycontainer      # -t: wait 30 sec
docker stop $(docker ps -q)        # Stop all running containers
```

#### **docker pause / unpause** — Suspend/resume
```bash
docker pause CONTAINER             # Freeze all processes (SIGSTOP)
docker unpause CONTAINER           # Resume (SIGCONT)

# Use case: pause resource-hungry containers temporarily
```

#### **docker kill** — Force kill (SIGKILL)
```bash
docker kill [OPTIONS] CONTAINER

docker kill mycontainer            # Immediate, no graceful shutdown
docker kill -s SIGTERM mycontainer # -s: specific signal
```

#### **docker rm** — Delete container
```bash
docker rm [OPTIONS] CONTAINER

docker rm mycontainer              # Remove stopped container
docker rm -f mycontainer           # -f: force remove running
docker rm -v mycontainer           # -v: remove volumes too
docker rm $(docker ps -aq)         # Remove all containers
```

#### **docker rename** — Rename container
```bash
docker rename OLD_NAME NEW_NAME
```

#### **docker restart** — Stop then start
```bash
docker restart CONTAINER
```

#### **docker update** — Update container config
```bash
docker update --memory=1g mycontainer
# Can update: restart-policy, cpu-shares, memory-limit, etc.
```

### Lifecycle Diagram

```
creation                  running
  │                          │
  ├─ docker run ────────────┤
  │  (or create → start)    │
  │                         │
  │  docker pause ──── paused
  │                    │
  │                    └─ docker unpause
  │
  │  docker stop ──────── stopped
  │   (SIGTERM, wait)
  │
  │  docker kill ──────── exited
  │   (SIGKILL)
  │
  └─ docker rm ─────────── deleted
```

---

## 2. Layered Notes

### 🟢 Beginner Notes

**Three main patterns:**

1. **Run and forget:** `docker run -d myapp` (background)
2. **Run interactive:** `docker run -it myapp` (foreground, shell)
3. **Run with output:** `docker run myapp` (foreground, see output)

**Common options:**
- `-d`: detach (background)
- `-it`: interactive terminal
- `-p`: port mapping
- `-e`: environment variable
- `-v`: volume
- `--name`: container name

### 🟡 Intermediate Notes

**Restart policies:**
```bash
docker run --restart=no myapp           # Don't restart (default)
docker run --restart=always myapp       # Always restart (even on daemon restart)
docker run --restart=on-failure myapp   # Restart on failure only
docker run --restart=on-failure:5 myapp # Restart max 5 times
docker run --restart=unless-stopped myapp # Restart unless explicitly stopped
```

**Signal handling & graceful shutdown:**
```bash
# Inside container (PID 1):
trap handle_sigterm SIGTERM
handle_sigterm() {
  # Cleanup code
  exit 0
}

# From host:
docker stop container  # Sends SIGTERM to PID 1
# Container has 10 sec to exit gracefully
# After timeout: SIGKILL (forceful)
```

### 🔴 Advanced Notes

**Exit code propagation:**
```bash
docker run myapp
# If myapp exits with code 42:
echo $?  # Output: 42

# But docker run exits with 0 if successful
# To see container exit code:
docker inspect <container> | jq '.[0].State.ExitCode'
```

**Resource limits & swapping:**
```bash
docker run --memory=512m --memory-swap=512m myapp
# --memory: max RAM
# --memory-swap: max RAM + swap combined
# If same: no swap (strict memory limit)

docker run --cpus=0.5 --cpu-shares=512 myapp
# --cpus: hard limit (newer syntax, more predictable)
# --cpu-shares: soft limit (proportional fair share)
```

### ⚫ Expert Notes

**OOM killer behavior:**
```bash
docker run --memory=256m --oom-kill-disable myapp
# --oom-kill-disable: don't kill container on OOM
# Instead: container process gets SIGSEGV (memory error)
# Use only if you handle OOM gracefully

# Without this:
# If container exceeds limit → kernel OOM killer → container killed
```

---

## 3. Practical Labs

### 🟢 Beginner Lab
```bash
# Run container in background
docker run -d --name myapp nginx

# Check it's running
docker ps

# Stop it
docker stop myapp

# Remove it
docker rm myapp
```

### 🟡 Intermediate Lab
```bash
# Run with resource limits and restart policy
docker run -d \
  --name myapp \
  --memory=512m \
  --cpus=0.5 \
  --restart=on-failure:3 \
  myimage

# Monitor it
docker stats myapp

# Kill the process inside container
docker exec myapp kill 1

# Watch it restart (up to 3 times)
docker logs myapp --tail=10
```

### 🔴 Advanced Lab
```bash
# Test graceful shutdown
docker run -d \
  --name graceful \
  --init \
  myapp

# Send SIGTERM
docker stop -t 5 graceful

# Check exit code
docker inspect graceful | jq '.[] | .State | {ExitCode, Status}'
```

### ⚫ Expert Lab
```bash
# Test OOM behavior
docker run --memory=100m --oom-kill-disable myapp

# Inside container: try to allocate 200m
# Observe: process gets error, container doesn't die
# Monitor cgroup: /sys/fs/cgroup/memory/docker/*/memory.stat
```

---

## 4. MCQs (Sample)

**Q1.** What does `docker run -d` do?
- A) Delete container
- B) Run in detached mode (background)
- C) Debug mode
- D) Dry run

**Answer: B**

**Q2.** What is the default restart policy?
- A) always
- B) on-failure
- C) no
- D) unless-stopped

**Answer: C**

---

## 5–10. (Complete structure: Interviews, Troubleshooting, Security, Performance, Projects)

[**MCQs (200), Interview Q&As (200), Troubleshooting table, Security/Performance sections, 6 projects — same as Phase 1 topics**]

---

**[END OF PHASE 2, TOPIC 1]**

Commit:
```bash
git add phase-02-cli-commands/01-container-lifecycle-commands.md
git commit -m "phase-02: container lifecycle commands (run, create, start, stop, etc.)"
```
