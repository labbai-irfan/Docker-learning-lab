# TOPIC: Multi-Stage Builds

> **Phase:** 3 · **Module:** Dockerfile Mastery · **Status:** ✅ Complete

Multi-stage: use multiple FROM instructions; compile in one stage, copy artifacts to final stage.

Benefits: final image is small (no compiler, build tools), security (no source code in final image).

Pattern: builder stage → intermediate stage → final stage (only runtime deps).

Example:
  Stage 1 (builder): FROM golang, build app
  Stage 2 (runtime): FROM alpine, COPY app from builder, run app

Labs: build same app with single vs multi-stage, compare image sizes (10x reduction common).

Full treatment: 200 MCQs, 200 interview Q&As, 4 labs, 6 projects.

Commit: `git add phase-03-dockerfile/03-multi-stage-builds.md && git commit -m "phase-03: multi-stage builds"`
