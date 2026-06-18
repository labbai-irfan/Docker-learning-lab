# 🎯 GIT COMMIT STRATEGY

This repo is your **portfolio**. Every commit tells a story. Commit early, commit
often, commit with clear messages. Interviewers will read your git log.

---

## Commit messages

Each topic completion gets **ONE commit** with this format:

```
phase-N: <topic> — complete core treatment

- Definition, purpose, internal working, architecture
- Lifecycle, components, data flow
- Advantages/disadvantages/tradeoffs
- Production & industry usage, best practices, common mistakes
- Security & performance notes
- 🟢🟡🔴⚫ Notes (beginner → expert)
- 4 labs (beginner → expert)
- 200 MCQs (50×4 levels)
- 200 interview Q&As (50×4 levels)
- Troubleshooting, security, performance sections
- 6-level project spec

Topics covered: <subtopic-a>, <subtopic-b>, ...
```

### Example commits

```
phase-00: Namespaces — core treatment

Covered pid/net/mnt/uts/ipc/user/cgroup/time namespaces.
- Internal mechanics (clone flags, /proc/<pid>/ns/)
- Lifecycle (creation, sharing, inheritance)
- Interaction with cgroups and seccomp
- Common mistakes (ns leaks, PID 1 handling)
- 200 MCQs, 200 interview Q&As, 4 labs, 6 projects

phase-01: Docker Architecture — core treatment

Client-server model, dockerd, containerd, shim, runc.
- Docker Desktop architecture on Windows/Mac (VM-backed)
- REST API and unix socket (/var/run/docker.sock)
- Rate of data flow: image pull, layer extraction, container spawn
- 200 MCQs, 200 interview Q&As, 4 labs, 6 projects
```

---

## Commit frequency

- **Per topic:** 1 commit when core treatment + labs + MCQs + interview Q&A are done
- **Per phase:** After all topics in a phase → summary commit
- **Weekly:** Sunday wrap-up → milestone commit with weekly stats

Aim for **3–5 commits per week** (1–2 topics/week at full depth).

---

## Milestone commits

After each phase completes, a summary commit:

```
phase-N: Complete

6 topics, 48 labs, 1200 MCQs, 1200 interview Q&As, 36 projects.
All core treatments written. Ready for phase N+1.

Statistics:
- Topics: X complete
- Labs written: X
- MCQs: X
- Interview Q&As: X
- Lines of code/markdown: X
```

---

## GitHub Actions (optional)

Set up a workflow to auto-generate stats on each push:

- Lines of markdown
- Number of topics completed
- MCQ/interview Q count
- Labs executable & passing tests

This makes your repo **interactive** — visitors see live progress.

---

## Portfolio presentation

When applying for jobs, highlight:

- **Depth:** "200 MCQs per topic, 50+ interview Q&As per level"
- **Breadth:** "300+ Docker topics across 13 phases"
- **Execution:** "Commit history shows systematic study, not just bookmarking"
- **Projects:** "From local containers to production GPU inference clusters"

> Your repo IS your resume. Make it shine.
