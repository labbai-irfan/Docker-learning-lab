# 🚀 Quick Start

You have just scaffolded the **Docker University** curriculum. Here's how to use it.

---

## 1. Repository is ready

✅ **Created:**

```
Docker-learning-lab/
├── README.md                          # Main index
├── CURRICULUM.md                      # Full knowledge tree
├── PROGRESS.md                        # Track your progress
├── JOURNAL.md                         # Learning journal
├── ROADMAP.md                         # 16-week study plan
├── COMMIT-STRATEGY.md                 # How to build your portfolio
├── _templates/                        # Topic/lab/MCQ/interview templates
│   ├── TOPIC_TEMPLATE.md
│   ├── LAB_TEMPLATE.md
│   ├── MCQ_TEMPLATE.md
│   └── INTERVIEW_TEMPLATE.md
├── phase-00-foundations/              # Phase 0 folder (empty README)
├── phase-01-docker-core/              # Phase 1 folder (empty README)
├── phase-02-cli-commands/             # Phase 2 folder
├── ... (phases 3–13 with READMEs)
├── labs/                              # Runnable lab code (scaffold only)
└── .gitignore
```

---

## 2. Initialize Git

```bash
cd Docker-learning-lab
git init
git add .
git commit -m "docker-university: scaffold all 13 phases, templates, and learning materials

- 13 phase folders with README module indexes
- Full knowledge tree: 300+ topics to cover
- Templates for topic treatment (notes, labs, MCQs, interview Q&As)
- Progress tracker, learning journal, roadmap
- Commit strategy & portfolio guide
- Labs directory structure for 250+ hands-on exercises

Ready to begin Phase 0: Foundations."
```

---

## 3. Start Phase 0

```bash
cd phase-00-foundations/
# Open 01-computing-fundamentals.md (doesn't exist yet — create it using _templates/TOPIC_TEMPLATE.md)
```

**Copy template and fill in:**

```bash
cp ../_templates/TOPIC_TEMPLATE.md 01-computing-fundamentals.md
# Edit and complete all 10 sections
```

---

## 4. Work top-to-bottom within a phase

For **each topic**:

1. **Read the notes** — 🟢 Beginner → 🟡 Intermediate → 🔴 Advanced → ⚫ Expert
2. **Do the labs** — beginner, intermediate, advanced, expert
3. **Take MCQs** — 50 per level; aim for 45+/50 before moving on
4. **Answer interview Q&As** — practice out loud; tie to concepts
5. **Build the project** — from beginner to production scope
6. **Log it in JOURNAL.md** — what you learned, what tripped you up
7. **Check it off in PROGRESS.md**
8. **Commit to git** — use the commit strategy template

---

## 5. Pacing

- **Full-depth treatment per topic:** ~5–8 hours
  - Notes: 1–2 hrs
  - Labs: 1–2 hrs
  - MCQs: 1 hr
  - Interview Q&As: 1 hr
  - Project: 2–3 hrs

- **Per phase:** ~30–40 hours (6 topics)
- **Full curriculum:** ~13 phases × 30–40 hrs = **~400–500 hours**
  - At 20 hrs/week = **5–6 months**
  - At 10 hrs/week = **10–12 months**

See [ROADMAP.md](ROADMAP.md) for a suggested 16-week sprint.

---

## 6. Track progress

Every Friday:
- Update [PROGRESS.md](PROGRESS.md) — mark topics complete
- Update [JOURNAL.md](JOURNAL.md) — week's reflections
- Update [ROADMAP.md](ROADMAP.md) — next week's plan
- Run `git log --oneline` to see your commit streak

---

## 7. Build your portfolio

Commit **after each topic** (not each phase). Your git log should show:

```
phase-01: Docker architecture — complete core treatment
phase-01: Images — complete core treatment
phase-01: Containers — complete core treatment
...
```

By week 16, you'll have **100+ commits** documenting systematic mastery.

Recruiters will see a **portfolio, not just a bookmark list.**

---

## 8. Key files to read first

1. [README.md](README.md) — what this curriculum is
2. [CURRICULUM.md](CURRICULUM.md) — the full knowledge tree
3. [_templates/TOPIC_TEMPLATE.md](_templates/TOPIC_TEMPLATE.md) — how to write a topic
4. [ROADMAP.md](ROADMAP.md) — suggested pacing (16 weeks)
5. [COMMIT-STRATEGY.md](COMMIT-STRATEGY.md) — how to build your portfolio

---

## 9. What to do right now

```bash
# 1. Read the curriculum tree
cat CURRICULUM.md | less

# 2. Start Phase 0
cd phase-00-foundations
# Pick topic 01-computing-fundamentals
# Copy the template
cp ../_templates/TOPIC_TEMPLATE.md 01-computing-fundamentals.md

# 3. Commit your first topic when done
git add phase-00-foundations/01-computing-fundamentals.md
git commit -m "phase-00: computing fundamentals — complete core treatment"

# 4. Move to the next topic
```

---

## ✅ Success metrics

- [ ] Scaffold complete (✅ done)
- [ ] Week 1: Phase 0, topics 1–2 done
- [ ] Week 4: Phases 0–1 complete, ready for Docker CLI
- [ ] Week 8: Phases 0–4 complete, can troubleshoot networks/storage
- [ ] Week 12: Phases 0–8 complete, ready for security interviews
- [ ] Week 16: All 13 phases complete, 100+ commits, ready for jobs
- [ ] Month 5: DCA certified, Docker Engineer on your resume
- [ ] Month 6: Job offer as Docker/DevOps/Platform Engineer

---

**Let's go. Start with Phase 0.**
