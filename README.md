# 🐳 Docker University — The Complete Mastery Curriculum

> A self-contained, university-grade Docker program. Built to take you from
> "what is a container?" to shipping, securing, observing, and scaling
> containerized systems in production — including AI/GPU infrastructure.
>
> After completing this lab you should be employable as a **Docker Engineer,
> DevOps Engineer, Platform Engineer, Cloud Engineer, Kubernetes Engineer,
> AI Infrastructure Engineer, or Site Reliability Engineer.**

---

## 📚 How this repository works

This repo IS the curriculum. Everything lives on disk so you can `git commit`
your progress and use it as a public **GitHub portfolio**.

```
Docker-learning-lab/   ← you are here
├── README.md              Master index (this file)
├── CURRICULUM.md          The complete Docker Knowledge Tree (13 phases)
├── PROGRESS.md            Track every topic, lab, MCQ set, project
├── JOURNAL.md             Daily/weekly learning journal
├── ROADMAP.md             Milestones, weekly plan, certification timeline
├── COMMIT-STRATEGY.md     How to commit & build the portfolio
├── _templates/            The "full spec" treatment applied to every topic
├── phase-00 … phase-13/   One folder per phase; READMEs are module indexes
└── labs/                  Runnable hands-on lab code
```

### Each TOPIC gets the same standardized treatment

Every topic node in the knowledge tree is written up using
[`_templates/TOPIC_TEMPLATE.md`](_templates/TOPIC_TEMPLATE.md), which covers:

`Definition · Purpose · Internal Working · Architecture · Lifecycle ·
Components · Data Flow · Advantages · Disadvantages · Tradeoffs · Production
Usage · Industry Usage · Best Practices · Common Mistakes · Security ·
Performance` → **Beginner / Intermediate / Advanced / Expert notes** →
**Labs (4 levels)** → **MCQs (50×4)** → **Interview Q&A (50×4)** →
**Troubleshooting · Security · Performance · Projects (6 levels)**.

---

## 🗺️ The 13 Phases

| # | Phase | Folder | Status |
|---|-------|--------|--------|
| 0 | Foundations (pre-Docker: kernel, namespaces, cgroups, OCI) | [phase-00-foundations](phase-00-foundations/) | 🔜 |
| 1 | Docker Core (architecture, images, containers, registries) | [phase-01-docker-core](phase-01-docker-core/) | 🔜 |
| 2 | Docker CLI & Commands (every command, every flag) | [phase-02-cli-commands](phase-02-cli-commands/) | 🔜 |
| 3 | Dockerfile Mastery (every instruction, BuildKit) | [phase-03-dockerfile](phase-03-dockerfile/) | 🔜 |
| 4 | Networking (drivers, DNS, proxies, service discovery) | [phase-04-networking](phase-04-networking/) | 🔜 |
| 5 | Storage & Data (volumes, mounts, distributed storage) | [phase-05-storage](phase-05-storage/) | 🔜 |
| 6 | Docker Compose (multi-container apps) | [phase-06-compose](phase-06-compose/) | 🔜 |
| 7 | Orchestration (Swarm + Kubernetes bridge) | [phase-07-orchestration](phase-07-orchestration/) | 🔜 |
| 8 | Security (scanning, signing, supply chain, runtime) | [phase-08-security](phase-08-security/) | 🔜 |
| 9 | Observability (metrics, logs, traces, alerts) | [phase-09-observability](phase-09-observability/) | 🔜 |
| 10 | CI/CD & Automation (Actions, GitLab, Jenkins, GitOps) | [phase-10-cicd](phase-10-cicd/) | 🔜 |
| 11 | Production Architecture (HA, DR, scaling, 12-factor) | [phase-11-production-architecture](phase-11-production-architecture/) | 🔜 |
| 12 | AI Infrastructure (GPU, vLLM, Ollama, RAG, agents) | [phase-12-ai-infrastructure](phase-12-ai-infrastructure/) | 🔜 |
| 13 | Capstone: Portfolio, Repo & Career | [phase-13-capstone-career](phase-13-capstone-career/) | 🔜 |

Legend: 🔜 not started · 🚧 in progress · ✅ complete

> Full topic breakdown: see [CURRICULUM.md](CURRICULUM.md).
> Track your own progress: see [PROGRESS.md](PROGRESS.md).

---

## 🚀 Getting started

1. Install Docker (see [phase-01-docker-core](phase-01-docker-core/) → Installation).
2. Read [CURRICULUM.md](CURRICULUM.md) end-to-end once to see the whole map.
3. Start at **Phase 0** and work down. Don't skip Foundations — it's *why*
   containers work, and it's where the hardest interview questions come from.
4. For every topic: read notes → do the labs → take the MCQs → answer the
   interview questions out loud → build the project → log it in
   [JOURNAL.md](JOURNAL.md) and check it off in [PROGRESS.md](PROGRESS.md).
5. Commit after every topic (see [COMMIT-STRATEGY.md](COMMIT-STRATEGY.md)).

---

## 🎯 Role outcomes

| Role | Phases that matter most |
|------|------------------------|
| Docker Engineer | 0–6 (everything core) |
| DevOps Engineer | 2,3,6,10 + 8,9 |
| Platform Engineer | 4,5,7,11 + 8,9,10 |
| Cloud Engineer | 1,4,5,7,11 |
| Kubernetes Engineer | 0,1,7 + 8,9 |
| AI Infrastructure Engineer | 1,3,5,12 + GPU |
| Site Reliability Engineer | 8,9,11 + 7,10 |

---

*Built with the Docker University curriculum generator. Star it, fork it, ship it.*
