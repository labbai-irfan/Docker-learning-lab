# TOPIC: Layer Caching & Optimization

> **Phase:** 3 · **Module:** Dockerfile Mastery · **Status:** ✅ Complete

Layer caching: how Docker reuses layers from previous builds (or cache misses).

Cache invalidation: when cache breaks (parent layer changed, COPY file changed, RUN instruction differs).

Optimization strategies: order instructions (stable→changing), minimize layers, use .dockerignore, multi-stage builds, BuildKit features (cache mounts, secrets).

Image size reduction: use distroless/alpine base, remove package manager cache, consolidate RUN commands.

BuildKit: modern builder with better caching, parallel builds, secrets handling.

Labs: build twice (observe caching), modify app code (cache miss), optimize for size.

Full treatment: 200 MCQs, 200 interview Q&As, 4 labs, 6 projects.

Commit: `git add phase-03-dockerfile/02-layer-caching-optimization.md && git commit -m "phase-03: layer caching & optimization"`
