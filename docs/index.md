---
hide:
  - toc
---

# Apic Applied AI Architect — Interview Prep

> A private, 12-module preparation system for the **Apic Applied AI Architect** role (Mumbai, Pre-Sales IC, India/APAC).
> Mirrors the structure of [`learngenai.sunmintz.com`](https://learngenai.sunmintz.com), adapted for interview-prep voice and Apic-bar grading.

---

## What this course is

This is not a generic Solutions Architect course. It is a hyper-specific, role-calibrated prep system that:

1. **Executes the 12-module syllabus** from the source meta-prompt (kept locally, not in repo)
2. **Layers in real Apic-specific calibration** — current Claude lineup (Opus 4.7, Sonnet 4.6, Haiku 4.5), real Apic research references (Constitutional AI, Responsible Scaling Policy, interpretability), real Indian BFSI context (RBI, DPDPA 2023, hyperscaler India regions)
3. **Grades your drill answers honestly** — Strong Hire / Hire / Lean No / Strong No, no softening
4. **Threads a single BFSI customer case study** through every module so architecture skills compound, not fragment
5. **Sequences the Claude API artifact build** into Module 04 — because zero hands-on Claude is the single largest credibility gap to close before applying

The course is **private prep, not for publishing**. Generic content will later be merged into a separate `sun-course-solarch` (AI Solutions Architect) course.

---

## The 12 modules

Each module: `Notes/` (concepts) + `Drills/` (graded practice) + `Case-Study/` (BFSI chapter) + `Interview-QA-Bank.md` (~26 questions). Module 04 adds `CodeLabs/`. Modules 05, 07, 08 add `SystemDesigns/`.

| # | Module | What this module tests |
|---|---|---|
| 01 | [Apic Calibration](01-Apic-Calibration/INDEX.md) | Whether you've internalized Apic's mission as operating mode, not vocabulary. The recruiter screen + initial values probe live here. |
| 02 | [Personal Narrative & Role-Fit](02-Personal-Narrative-and-Role-Fit/INDEX.md) | Whether your career arc tells a coherent story that lands you in this seat — and whether you own the IC lateral move without defensiveness. |
| 03 | [Customer Discovery & Pre-Sales](03-Customer-Discovery-and-Pre-Sales/INDEX.md) | Whether you can run a discovery call with a regulated enterprise and translate ambiguous business asks into shippable architecture. |
| 04 | [Claude Platform Architecture](04-Claude-Platform-Architecture/INDEX.md) | Whether you understand Claude's actual surface area — API, Claude for Work, Bedrock, Vertex, model selection, prompt caching, tool use. **Includes the GitHub artifact build.** |
| 05 | [Solution Architecture Patterns](05-Solution-Architecture-Patterns/INDEX.md) | Whether you can design enterprise-grade architectures for the six BFSI use cases — HLD, LLD, integration boundaries, failure modes. |
| 06 | [Evaluation Architecture](06-Evaluation-Architecture/INDEX.md) | Whether you treat evals as the production go/no-go gate they are — not as an afterthought. Apic weighs this heavily. |
| 07 | [RAG in Regulated Enterprises](07-RAG-in-Regulated-Enterprises/INDEX.md) | Whether you can architect retrieval that passes RBI scrutiny — chunking, embeddings, security boundaries, residency, hallucination control. |
| 08 | [Agentic & Tool-Use Architecture](08-Agentic-and-Tool-Use-Architecture/INDEX.md) | Whether you can design agentic systems with safe tool schemas, MCP integration, and human-in-the-loop approval — and explain why automation has limits. |
| 09 | [Security, Privacy, Governance](09-Security-Privacy-Governance/INDEX.md) | Whether a CISO would trust you. Prompt injection, data leakage, audit, role-based access, regulated-workload posture. |
| 10 | [Cost, Latency, Reliability](10-Cost-Latency-Reliability/INDEX.md) | Whether you treat production AI as a system, not a demo — observability, cost-per-BU, latency budgets, fallback strategy. |
| 11 | [Storytelling, Workshops, Demos](11-Storytelling-Workshops-Demos/INDEX.md) | Whether you can speak the same architecture in five registers — CEO, CIO, CTO, CISO, hands-on engineer — without losing technical credibility or business clarity. |
| 12 | [Full Loop Simulation](12-Full-Loop-Simulation/INDEX.md) | Whether the prep adds up. Mock recruiter → HM → onsite Loop 1 (technical) → onsite Loop 2 (values + project deep dive) — all graded. |

---

## How to use this course

**Sequential.** Modules build on each other. Module 04's CodeLabs require Module 03's discovery framing to make sense. Module 11's executive storytelling needs Modules 05–10's architecture depth as raw material.

**Drill-driven.** Every module ends with practice drills you actually perform — written, spoken aloud, or whiteboarded. Your answers get graded against a Strong Hire / Hire / Lean No / Strong No rubric. Grades + upgraded versions are logged in [Drill Tracker](Drill-Tracker.md).

**Case-study threaded.** A single Indian BFSI customer (six use cases, real constraints) appears in every module's `Case-Study/` chapter. By Module 12, you have a complete enterprise architecture portfolio for one customer — far more memorable in an interview than five disconnected toy examples.

**Apic-calibrated.** Every "good answer" example references actual Apic research, the actual JD language, the actual Claude model lineup. Generic AI talk gets a Lean No, by design.

---

## Application trigger checklist

You do **not** apply until all four are true:

- [ ] **Apic-tailored resume `.docx`** written and reviewed (Module 02 deliverable)
- [ ] **Claude API artifact** published on GitHub — Python SDK + tool use + structured output + prompt caching + custom eval harness (Module 04 deliverable)
- [ ] **"Tell me about yourself" + "Why Apic"** both at Strong Hire (Module 01 drills)
- [ ] **Clean lateral-move answer** — non-defensive, principal-grade narrative (Module 02 deliverable)

The fit analysis flagged "apply this week" — but speed matters less than these four boxes. The artifact alone is 4–6 hours of focused work.

---

## Sources

- **Meta-prompt** (the source of truth): kept locally, not committed
- **Job description**: kept locally, not committed
- **Fit analysis**: kept locally, not committed
- **Current resume**: kept locally, not committed
- **Public interview intel**: techprep.app + jobright.ai (covered SWE/RS/MLE; loop structure transfers, technical-round content does not)
- **Approved plan**: `vibes/plan.md` (in repo, outside the rendered docs site)
- **Repo context for future agents**: `vibes/CLAUDE.md`
- **Session handoff**: `vibes/handoff.md`
