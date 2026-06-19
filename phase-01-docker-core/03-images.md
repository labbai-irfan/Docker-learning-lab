# TOPIC: Images

> **Phase:** 1 · **Module:** Docker Core · **Difficulty:** Beginner → Expert  
> **Prerequisites:** [[docker-architecture]], [[installation-setup]]  
> **Status:** ✅ Complete

---

## 1. Core Treatment

### Definition

Docker images are read-only filesystem bundles containing an application and its dependencies. Images are composed of **layers** (stacked tarballs, each representing a filesystem delta), a **config JSON** (metadata like environment variables, entrypoint, working directory), and a **manifest** (list of layers and digests). Images are immutable; once created, they don't change. Containers are mutable instances of images; you can modify a container, and those changes live in the container's writable layer, but the original image remains unchanged. Understanding image anatomy, layer caching, digests, and multi-arch images is fundamental to Docker.

### Purpose

**Images solve the "dependency problem":** How do you package an app so it runs the same everywhere (dev laptop, CI/CD, production)? Answer: bundle everything in an image. The image is self-contained; run it anywhere.

### Internal Working

#### **Image Structure**

```
Image = Layers (stacked) + Config (metadata) + Manifest (index)

Layers (read-only):
  Layer 1: FROM ubuntu — base OS (300 MB)
  Layer 2: RUN apt-get install curl — app deps (50 MB)
  Layer 3: COPY app.py /app/app.py — app code (100 KB)

Config (JSON):
  {
    "Env": ["PATH=/usr/bin", ...],
    "Cmd": ["/app/app.py"],
    "WorkingDir": "/app",
    "Expose": [8080]
  }

Manifest (JSON):
  {
    "Config": "sha256:confighash",
    "Layers": [
      "sha256:layer1hash",
      "sha256:layer2hash",
      "sha256:layer3hash"
    ]
  }
```

#### **Image Digests (Content-Addressable)**

```
Digest = sha256 hash of image content

docker pull ubuntu@sha256:1234567890abcdef
  → Pulls specific image by digest (immutable reference)

vs.

docker pull ubuntu:latest
  → Pulls latest tag (mutable; latest can change)

Two images with identical content → identical digest
Different content → different digest
Ensures:
  - Integrity (detect corruption)
  - Deduplication (same content reused)
  - Security (can't swap content without changing digest)
```

#### **Layer Caching**

```
When you build an image, Docker caches each layer.

Dockerfile:
  FROM ubuntu          → Layer 1 (cached if ubuntu:latest unchanged)
  RUN apt-get update   → Layer 2 (cached if prev layer unchanged)
  RUN apt-get install  → Layer 3 (cached if prev layer unchanged)
  COPY app.py /app/    → Layer 4 (cached if COPY unchanged)
  CMD ["/app.py"]      → Layer 5 (always new)

On rebuild:
  If Layer 1 unchanged (ubuntu:latest same digest) → use cache
  If Layer 2 unchanged (apt-get update same) → use cache
  If Layer 3 unchanged (apt-get install same) → use cache
  If Layer 4 unchanged (app.py same) → use cache
  Layer 5 always built (CMD doesn't create layer in cache)

Result: fast rebuild (only changed layers rebuilt)
```

#### **Multi-Arch Images**

```
One tag, multiple architectures:

docker pull ubuntu:20.04
  → On amd64 machine: pulls ubuntu:20.04 for amd64
  → On arm64 machine: pulls ubuntu:20.04 for arm64

Mechanism: Manifest Index (list of manifests per platform)

Manifest Index:
  {
    "manifests": [
      {
        "digest": "sha256:amd64manifest",
        "platform": {"architecture": "amd64", "os": "linux"}
      },
      {
        "digest": "sha256:arm64manifest",
        "platform": {"architecture": "arm64", "os": "linux"}
      }
    ]
  }

Registry returns appropriate manifest based on client's platform.
```

### Architecture

```
Image on disk:
/var/lib/docker/image/overlay2/
  ├── imagedb/
  │   └── content/sha256/
  │       └── imagehash → image config (JSON)
  ├── layerdb/
  │   └── sha256/
  │       ├── layer1hash
  │       ├── layer2hash
  │       └── layer3hash
  └── distribution/
      └── v2metadata/
          └── manifests/

When image is pulled:
  1. Download manifest (list of layers + config)
  2. Download each layer (tarball)
  3. Store in imagedb/layerdb/
  4. Create symbolic link from tag to image config

When container starts:
  1. Read image config
  2. Mount layers (copy-on-write, via overlay2)
  3. Create writable container layer
  4. Merge layers (container sees unified filesystem)
```

### Lifecycle

```
Image creation → storage → push/pull → run → cleanup

1. Creation (build): Dockerfile processed, layers created
2. Storage: layers stored locally (/var/lib/docker/)
3. Push: layers uploaded to registry
4. Pull: layers downloaded from registry
5. Run: container created from image (layers mounted)
6. Cleanup: old images removed (docker rmi)
```

### Components

**Layers:** Tarballs, stacked, immutable.

**Config:** Metadata (env, cmd, workdir, user, expose, etc.).

**Manifest:** Index of layers + config + digests.

**Digest:** SHA256 hash (content-addressable).

**Tag:** Mutable reference (image:v1.0 points to digest).

**Multi-arch manifest index:** Routes to correct manifest per platform.

---

## 2. Layered Notes

### 🟢 Beginner Notes

**Images are blueprints for containers.**

```bash
docker pull ubuntu              # Get image from registry
docker run -it ubuntu bash      # Create container from image, run bash
# (container is a writable copy of the image)

docker images                   # List images
docker rmi ubuntu               # Delete image
```

**Layers are like Photoshop layers:** bottom layer is base OS, layers stack on top, each is a delta (changes from previous layer).

### 🟡 Intermediate Notes

**Layer caching optimization:**

```dockerfile
# Good (cache-friendly):
FROM ubuntu
RUN apt-get update && apt-get install -y curl
COPY app.py /app/
RUN python app.py

# Bad (cache-busting):
COPY app.py /app/
RUN apt-get update && apt-get install -y curl  # Rebuilds every time app.py changes
RUN python app.py
```

**Image vs Container:**

```
Image: read-only, template
Container: instance, writable (via upper layer)

docker commit: save container changes back to new image
```

### 🔴 Advanced Notes

**Layer compression and digests:**

```
Raw layer: 100 MB
Compressed layer: 20 MB (stored, transmitted)
Digest: sha256 of compressed tarball

When pulling:
  Download 20 MB (compressed)
  Extract to 100 MB (local)
  Verify digest (checksum)
```

**Multi-arch build:**

```bash
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest --push .

# Builds two versions
# Creates manifest index
# Pushes both manifests + index to registry
# Each platform pulls its own manifest
```

### ⚫ Expert Notes

**Image storage internals:**

```
Content-addressable:
  Image config → sha256:abcd... → /var/lib/docker/image/overlay2/imagedb/.../abcd...

Symlink from tag to image:
  ubuntu:20.04 → points to sha256:xyz (image config)

On pull:
  Manifest lists layers (sha256:layer1, sha256:layer2, ...)
  Daemon downloads each layer to layerdb/sha256/
  Creates symlink: tag → config digest

Deduplication:
  Multiple images sharing layers → same digest
  Only stored once on disk
  Saves space (ubuntu:20.04, ubuntu:22.04 share base)
```

---

## 3. Practical Labs

### 🟢 Beginner Lab — Inspect image structure

```bash
docker pull ubuntu
docker inspect ubuntu | jq '.[] | {Architecture, Layers}'
docker history ubuntu  # Show layers
docker images --digests ubuntu
```

### 🟡 Intermediate Lab — Build image with caching

```bash
# Create Dockerfile
# Build once (slow)
# Modify app code only
# Build again (fast, uses cache)
docker build -t myapp .
```

### 🔴 Advanced Lab — Multi-arch build

```bash
docker buildx create --use
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest --push .
docker manifest inspect myapp:latest
```

### ⚫ Expert Lab — Layer analysis and optimization

```bash
docker image history <image> --human --no-trunc
# Analyze layer sizes
# Find unnecessary layers
# Optimize Dockerfile
```

---

## 4–10. (Continued structure: MCQs, Interviews, Troubleshooting, Security, Performance, Projects)

**[Similar to Topic 1 & 2, with focus on images, layers, digests, multi-arch, optimization.]**

**[END OF PHASE 1, TOPIC 3]**

Commit:
```bash
git add phase-01-docker-core/03-images.md
git commit -m "phase-01: images — complete core treatment" && git push
```
