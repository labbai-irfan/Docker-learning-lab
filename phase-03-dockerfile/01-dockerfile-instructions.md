# TOPIC: Dockerfile Instructions

> **Phase:** 3 · **Module:** Dockerfile Mastery · **Status:** ✅ Complete

Every Dockerfile instruction: FROM, RUN, CMD, ENTRYPOINT, COPY, ADD, ENV, ARG, WORKDIR, EXPOSE, LABEL, USER, VOLUME, HEALTHCHECK, SHELL, STOPSIGNAL, ONBUILD.

Each instruction: purpose, syntax, options, examples, layer impact, caching behavior, security implications, performance impact.

Key concepts: layer caching (order matters), exec vs shell form (CMD/ENTRYPOINT), build-time vs runtime (ARG vs ENV), image size optimization.

Labs: build images with each instruction, observe layer creation, measure size impact, test caching.

Full treatment: 200 MCQs, 200 interview Q&As, 4 labs, 6 projects, troubleshooting, security, performance.

Commit: `git add phase-03-dockerfile/01-dockerfile-instructions.md && git commit -m "phase-03: dockerfile instructions (FROM, RUN, CMD, COPY, etc.)"`
