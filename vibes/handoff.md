# Session handoff

> **Read root `CLAUDE.md` first** for repo context, then this file for current
> state. Read `vibes/plan.md` for the full approved plan. Read `vibes/Build-Plan.md`
> for the 6-week content schedule and status flags per file.

---

## Where we are

**Date:** 2026-05-03
**Status:** Session 1 complete. Session 2 opened.
**Application status:** **Not yet applied.** All 4 checklist items still open.

---

## Session 1 decisions (2026-04-30)

The candidate made four decisions via AskUserQuestion before plan approval:

| Question | Answer |
|---|---|
| Subdomain | `learnapic.sunmintz.com` (CNAME set, not published) |
| Visibility | **Private prep only.** Will later merge generic content into `sun-course-solarch` |
| Session 1 scope | Scaffold + Module 01 + run the 3 drills |
| Coaching style | **Brutally honest, Apic-bar calibrated.** Strong Hire / Hire / Lean No / Strong No. No softening. |

Key calibration from research (techprep.app, jobright.ai):
- Apic loop: recruiter → HM → 90-min CodeSignal → onsite Loop 1 (technical) → onsite Loop 2 (values + 20-min project deep dive + 40-min skeptical Q&A) → references
- Behavioral/values rounds carry **equal weight** to technical
- Mission alignment requires references to **specific Apic work** (Constitutional AI, RSP, interpretability)
- Project deep dive is reviewed "as if in a design review with a skeptical senior engineer"
- All articles cover SWE/RS/MLE roles — **none cover Applied AI / Pre-Sales / Solutions Architect.** The loop structure transfers; the technical-round content does not. The meta-prompt fills that gap.

---

## Session 2 decisions (2026-05-03)

| Change | Decision |
|---|---|
| Planning docs location | Move all planning `.md` files out of `docs/` into `vibes/`. `docs/Build-Plan.md` → `vibes/Build-Plan.md`. Removed from MkDocs nav. |
| CLAUDE.md root | Updated to reflect accurate module status (Module 01 Done, 02–12 Outline scaffolds). |
| handoff.md | This rewrite. |

---

## What's done

### Infrastructure (Session 1)
- [x] `vibes/plan.md` — verbatim copy of approved plan
- [x] `vibes/CLAUDE.md` — repo context for future agents
- [x] `vibes/handoff.md` — session state (this file)
- [x] `mkdocs.yml`, `CNAME`, `README.md`, `.gitignore`, `LICENSE`
- [x] `docs/index.md` — 12-module syllabus landing page
- [x] `docs/overrides/main.html` — copied + COURSE label updated to "Apic Prep"
- [x] `docs/stylesheets/extra.css` — `.personal` and `.grade` admonition styles added
- [x] All 12 module folders created with placeholder `INDEX.md`
- [x] `mkdocs build --strict` verified — no broken nav references

### Module 01 — Apic Calibration (Session 1) — **Done**
- [x] `INDEX.md`
- [x] `Notes/01-Mission-Values-Constitutional-AI.md`
- [x] `Notes/02-Role-Decoded-Applied-AI-Architect.md`
- [x] `Notes/03-Interview-Loop-Map.md`
- [x] `Notes/04-What-Good-vs-Weak-Looks-Like.md`
- [x] `Notes/05-Lateral-Move-and-Mission-Alignment.md`
- [x] `Drills/01-Tell-Me-About-Yourself.md`
- [x] `Drills/02-Why-Apic.md`
- [x] `Drills/03-Customer-Architecture-Story.md`
- [x] `Case-Study/01-BFSI-Customer-Intro.md`
- [x] `Interview-QA-Bank.md` — 26 questions, graded

### Modules 02–12 (Session 1) — **Outline**
- [x] Full folder scaffold for each module: `INDEX.md`, `Notes/` (5 files), `Drills/` (3 files), `Case-Study/` (1 file), plus `CodeLabs/` (Mod 04) and `SystemDesigns/` (Mods 05, 07, 08)
- [x] Each file has a structured H2/H3 outline + section briefs — body content is the 6-week build task

### Cross-module (Session 1)
- [x] `docs/Drill-Tracker.md` — grading log scaffolded; entries pending
- [x] `docs/Interview-Questions/INDEX.md` — aggregator scaffolded
- [x] `docs/Interview-Questions/01-Apic-Calibration.md` — Module 01 Q&A links

### Session 2 changes
- [x] `docs/Build-Plan.md` moved to `vibes/Build-Plan.md`
- [x] `mkdocs.yml` — `Build Plan` nav entry removed
- [x] Root `CLAUDE.md` updated — accurate module status, Build-Plan.md location noted
- [x] This handoff rewritten with 2026-05-03 state

### Drills with the candidate — **NOT YET RUN**
- [ ] Drill 1: "Tell me about yourself" (2-min) — graded + logged
- [ ] Drill 2: "Why Apic" (90-sec) — graded + logged
- [ ] Drill 3: Customer architecture story — graded + logged

> Drills are interactive. Once run, grades append to `docs/Drill-Tracker.md` and upgrade examples land in `!!! personal` blocks in the relevant `Drills/*.md` files.

---

## What's pending

### Immediate (before moving to Module 02)
- [ ] Run the 3 Module 01 drills. Grade each against the rubric. Log to Drill-Tracker.md.
- [ ] Re-run any drill that scores Lean No or below until it reaches Hire or better.
- [ ] Confirm TMAY + Why Apic are both at Strong Hire before starting Module 02.

### Module 02–12 body content (6-week build)

See `vibes/Build-Plan.md` for the full sequence, status flags, and deliverables per week.

| Week | Module(s) | Hard deliverable |
|---|---|---|
| **Week 1** | 02 — Personal Narrative & Role-Fit | Apic-tailored `.docx` resume; clean lateral-move answer |
| **Week 2** | 03 — Customer Discovery · 04 — Claude Platform Architecture | Discovery checklist; **Claude API artifact on GitHub** |
| **Week 3** | 05 — Solution Architecture · 06 — Evals · 07 — RAG | 6 SystemDesigns + eval framework + 2 RAG designs |
| **Week 4** | 08 — Agentic · 09 — Security · 10 — Cost/Latency | CISO architecture review; cost/latency model for BFSI |
| **Week 5** | 11 — Storytelling, Workshops, Demos | Demo script; objection library; exec pitch |
| **Week 6** | 12 — Full Loop Simulation | 5 mock-round transcripts; final go/no-go |

### Out-of-course deliverables
- [ ] Apic-tailored resume `.docx` (separate file, not in this repo)
- [ ] Claude API artifact (separate GitHub repo, built per Module 04 CodeLabs)

---

## Application trigger checklist

**Do not apply until all four are true:**

- [ ] Apic-tailored resume `.docx` written and reviewed (Module 02, Week 1)
- [ ] Claude API artifact published on GitHub (Module 04 CodeLabs, Week 2)
- [ ] "Tell me about yourself" + "Why Apic" both at **Strong Hire**
- [ ] Clean, non-defensive lateral-move answer (Module 02 output)

---

## Next session pickup point

**Recommended next action sequence:**

1. Run the 3 Module 01 drills interactively (candidate speaks, Claude grades). Each drill: speak your answer → Claude grades → if Lean No or below, revise → re-grade → log the final grade in `docs/Drill-Tracker.md`.
2. Once TMAY + Why Apic are at Strong Hire → start Module 02 body content.
3. Module 02 before Module 04 CodeLabs — the lateral-move answer unblocks the application checklist fastest. The artifact is 4–6 hours of separate coding work; don't let it block the narrative work.

**Module 02 opening move:** Start with `Notes/02-Lateral-Move-Answer.md`. Get that to Strong Hire. Everything else in Module 02 flows from it.

---

## Open questions

- Should Module 04 CodeLabs include a full working repo *inside* this course, or just architecture notes pointing to a separate `sun-claude-artifact` repo? (Default: separate repo for the actual code; this course's CodeLabs hold the architecture, lab walkthroughs, and the README of the artifact.)
- Should Drill-Tracker grades be visible side-by-side with the original question, or kept in a separate log per drill date?
- Confirm: the eventual `sun-course-solarch` repo will receive only the generic note bodies (sans `!!! personal` blocks) — or will it also receive the Drills sub-folders (re-genericized)?

> Don't try to answer these in the middle of a working session. Park them here, raise them at session boundaries.
