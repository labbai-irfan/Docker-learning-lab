# TOPIC: <Topic Name>

> **Phase:** <n> · **Module:** <module> · **Difficulty:** <Beginner→Expert>
> **Prerequisites:** [[topic-a]], [[topic-b]]
> **Status:** 🔜 not started

Copy this file for every topic node in the knowledge tree. Fill **every**
section. A topic is not "done" until all sections below are complete and the
checkboxes in `PROGRESS.md` are ticked.

---

## 1. Core Treatment

### Definition
*What it is, in one precise paragraph. No hand-waving.*

### Purpose
*The problem it solves. Why it exists. What life is like without it.*

### Internal Working
*Mechanics under the hood. Syscalls, kernel features, data structures,
algorithms. Be concrete — name the actual components.*

### Architecture
*Diagram (ASCII or linked image) + description of the moving parts and how
they connect.*

### Lifecycle
*State machine / phases from creation to destruction.*

### Components
*Enumerate each sub-part and its responsibility.*

### Data Flow
*Trace a request/operation end-to-end through the system.*

### Advantages
### Disadvantages
### Tradeoffs
*Explicit "X buys you Y at the cost of Z" statements.*

### Production Usage
*How real systems use it in prod (not toy examples).*

### Industry Usage
*Who uses it, at what scale, for what — named patterns.*

### Best Practices
### Common Mistakes
### Security Considerations
### Performance Considerations

---

## 2. Layered Notes

### 🟢 Beginner Notes
*Plain-language mental model. Analogies allowed.*

### 🟡 Intermediate Notes
*Real configuration, flags, and the "gotchas" beginners hit.*

### 🔴 Advanced Notes
*Internals, edge cases, interaction with other subsystems.*

### ⚫ Expert Notes
*Source-level behavior, kernel/runtime details, scale considerations,
contributing-to-the-project depth.*

---

## 3. Practical Labs

> See [`LAB_TEMPLATE.md`](LAB_TEMPLATE.md). Each lab: objective → steps →
> expected output → verification → cleanup.

- [ ] **Beginner Lab** — <one line>
- [ ] **Intermediate Lab** — <one line>
- [ ] **Advanced Lab** — <one line>
- [ ] **Expert Lab** — <one line>

---

## 4. MCQs
> See [`MCQ_TEMPLATE.md`](MCQ_TEMPLATE.md). 50 per level, answers + explanations.

- [ ] 50 Beginner MCQs
- [ ] 50 Intermediate MCQs
- [ ] 50 Advanced MCQs
- [ ] 50 Expert MCQs

---

## 5. Interview Questions (with answers)
> See [`INTERVIEW_TEMPLATE.md`](INTERVIEW_TEMPLATE.md). 50 per level.

- [ ] 50 Beginner Q&A
- [ ] 50 Intermediate Q&A
- [ ] 50 Advanced Q&A
- [ ] 50 Expert Q&A

---

## 6. Troubleshooting
| Symptom / Error | Root Cause | Debugging Process | Fix |
|-----------------|-----------|-------------------|-----|
| | | | |

**Production incidents seen in the wild:**
- *Incident → blast radius → detection → remediation → prevention.*

---

## 7. Security
- **Risks:**
- **Vulnerabilities (CVE classes):**
- **Attack Vectors:**
- **Hardening:**
- **Compliance mapping (CIS / PCI / HIPAA / SOC2):**

---

## 8. Performance
- **Key Metrics:**
- **Bottlenecks:**
- **Optimization techniques:**
- **Scaling strategy (horizontal / vertical):**

---

## 9. Projects
- [ ] **Beginner Project** —
- [ ] **Intermediate Project** —
- [ ] **Advanced Project** —
- [ ] **Expert Project** —
- [ ] **Production Project** —
- [ ] **Enterprise Project** —

---

## 10. Self-Assessment
*Can I explain this to a beginner? Can I defend it in a senior interview?
Can I debug it at 3am in prod? If not, which section do I re-read?*
