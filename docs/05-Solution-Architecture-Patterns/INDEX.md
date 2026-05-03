# 05 — Solution Architecture Patterns

**Overview:** Design patterns for AI systems in enterprise. You'll learn the anatomy of a reference architecture, where to draw integration boundaries, how to model cost and latency, and what can go wrong.

---

## What This Module Tests

1. **Architecture depth** — Can you design a system that's credible at a whiteboard?
2. **Trade-off reasoning** — Can you defend a design choice when questioned?
3. **Failure mode thinking** — What breaks? What's the blast radius? How do you mitigate?
4. **Pattern recognition** — Can you identify patterns across use cases?

---

## Learning Path

**Recommended order:**

| Step | File | Purpose | Time |
|------|------|---------|------|
| 1 | [01-Reference-Architecture-Anatomy](Notes/01-Reference-Architecture-Anatomy.md) | Components: LLM, retrieval, evaluation, orchestration, HITL | 15 min |
| 2 | [02-Integration-Boundary-Doctrine](Notes/02-Integration-Boundary-Doctrine.md) | Where does the AI system end? Where does the customer's system begin? | 15 min |
| 3 | [03-Failure-Modes-Catalog](Notes/03-Failure-Modes-Catalog.md) | Hallucination, latency, cost explosion, data leak, compliance violation | 15 min |
| 4 | [04-Cost-and-Latency-Modeling-Primer](Notes/04-Cost-and-Latency-Modeling-Primer.md) | Math: token count → cost; latency SLA → model choice | 15 min |
| 5 | [05-Rollout-Patterns](Notes/05-Rollout-Patterns.md) | Shadow mode, canary, gradual rollout, HITL gates | 10 min |

**Total notes time:** ~70 minutes

---

## Practice Exercises

| Drill | Focus | Time | Rubric |
|------|-------|------|--------|
| [01-Whiteboard-Support-Agent-Assist](Drills/01-Whiteboard-Support-Agent-Assist.md) | Design a customer support agent on the board; walk through the flow | 15 min | Clarity, completeness, realistic constraints |
| [02-Cross-Architecture-Pattern-Recognition](Drills/02-Cross-Architecture-Pattern-Recognition.md) | Given 3 use cases (search, agent, classification), identify the patterns | 10 min | Abstraction, pattern clarity |
| [03-Trade-off-Defense](Drills/03-Trade-off-Defense.md) | "Why Sonnet and not Haiku?" "Why not just LangChain?" Defend design choices | 10 min | Logic, honesty, customer awareness |

**Goal:** All 3 at **Hire** level minimum. **Strong Hire** means you never say "YOLO" in an architecture.

---

## System Designs

Production-grade architecture documents for all 6 BFSI use cases:

- [01-Customer-Support-Agent-Assist](SystemDesigns/01-Customer-Support-Agent-Assist.md) — HLD/LLD for support use case
- [02-Internal-SOP-Policy-Search](SystemDesigns/02-Internal-SOP-Policy-Search.md) — RAG system for policy lookup
- [03-Relationship-Manager-Copilot](SystemDesigns/03-Relationship-Manager-Copilot.md) — Context-aware customer relationship system
- [04-Compliance-Document-Review](SystemDesigns/04-Compliance-Document-Review.md) — Regulatory compliance automation
- [05-Developer-Productivity-with-Claude-Code](SystemDesigns/05-Developer-Productivity-with-Claude-Code.md) — Internal developer productivity
- [06-Executive-Analytics-Assistant](SystemDesigns/06-Executive-Analytics-Assistant.md) — Analytics & business intelligence

---

## Running BFSI Customer

- **[01-Six-Use-Case-Meta-Pattern](Case-Study/01-Six-Use-Case-Meta-Pattern.md)** — BFSI customer has 6 AI use cases. What patterns do they share? Where are the differences?

---

## Knowledge Check

**Interview-QA-Bank** covers:
- Architecture design (technical questions)
- Trade-off reasoning (HM/technical questions)
- Failure mitigation (values/technical questions)

👉 **Skim the Q&A Bank first.** Mark your 5 weakest questions.

**[Interview-QA-Bank](Interview-QA-Bank.md)**

---

## How to Work This Module

1. ✅ **Read all 5 Notes in order** (~70 minutes)
2. ✅ **Run the 3 Drills** — Get comfortable at the whiteboard
3. ✅ **Study the 6 System Designs** — These are production reference documents
4. ✅ **Review the case study** — Understand the BFSI customer's 6-use-case landscape
5. ✅ **Skim Interview-QA-Bank** — Mark your 5 weakest questions
6. ✅ **Mark complete** when all Drills reach **Hire** level

**Strong-Hire bar:** You can sketch a reference architecture and articulate failure modes under skeptical questioning.

---

## What's Next

After this module:
- You can design a credible enterprise AI system
- You know where to draw integration boundaries
- You can model cost and latency confidently
- You're prepared for whiteboard design interviews

**Next Module:** [06-Evaluation-Architecture](../06-Evaluation-Architecture/INDEX.md)

---

## Quick Reference

**Key Files:**
- Notes: [01-Reference-Architecture-Anatomy](Notes/01-Reference-Architecture-Anatomy.md) and 4 more
- Drills: [01-Whiteboard-Support-Agent-Assist](Drills/01-Whiteboard-Support-Agent-Assist.md) and 2 more
- System Designs: [01-Customer-Support-Agent-Assist](SystemDesigns/01-Customer-Support-Agent-Assist.md) and 5 more
- Case Study: [01-Six-Use-Case-Meta-Pattern](Case-Study/01-Six-Use-Case-Meta-Pattern.md)
- Interview Q&A: [Interview-QA-Bank](Interview-QA-Bank.md)
