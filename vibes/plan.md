# Apic Applied AI Architect — Interview Prep Course (MkDocs Material)

> **Verbatim copy of the approved Claude Code plan.** Source of truth at
> `~/.claude/plans/attached-is-a-dream-moonlit-moler.md`. Kept here so it lives
> with the source materials and survives session resets.

---

## Context

You are preparing for the **Apic Applied AI Architect** role (Mumbai, Pre-Sales IC, India/APAC) — your highest-fit role at 89%, your dream role. You have **not yet applied**. Today is **2026-04-30**. You have **zero hands-on Claude API experience** — the single largest credibility gap to close before applying.

This plan delivers a **MkDocs Material course** that mirrors your existing `sun-course-genai` site (`learngenai.sunmintz.com`) and executes the ChatGPT-authored meta-prompt — with my own Apic-specific calibration layered in.

**Final outcome of this prep arc**: pass the Apic interview loop. **Final outcome of the course artifact**: a structured prep system you can later merge into a generic *AI Solutions Architect* course (the empty `sun-course-solarch` repo).

---

## Confirmed decisions

| Decision | Value |
|---|---|
| Static-site stack | MkDocs Material — mirror `sun-course-genai` config (Inter + JetBrains Mono, custom palette, dark/light, mermaid, admonitions, code-copy) |
| Repo | Current course repository |
| Subdomain (deferred publish) | `learnapic.sunmintz.com` — set `site_url` and `CNAME` accordingly, but do **not** push to GitHub Pages |
| Visibility | **Private prep only** — no publish. Later merged into a generic `sun-course-solarch` course |
| Voice | Personal-prep, second-person, brutally honest grading on Strong Hire / Hire / Lean No / Strong No rubric |
| Merge-friendliness | Generic note bodies; all personal anchors and graded answers in distinctive `!!! personal` admonitions, easy to grep-and-strip later |
| Source-of-truth | The ChatGPT meta-prompt's 12 modules, kept as-is, with my Apic-specific upgrades layered throughout |
| AIOfferly article | Skipped (returned no content; we have enough from the other two + meta-prompt + JD) |

---

## Apic-specific upgrades layered onto the meta-prompt

1. **Real April-2026 Claude lineup** — Opus 4.7 (`claude-opus-4-7`), Sonnet 4.6 (`claude-sonnet-4-6`), Haiku 4.5 (`claude-haiku-4-5-20251001`).
2. **Real JD language verbatim** — "trusted technical advisor," "paint the vision," "develop evals," "common integration patterns."
3. **Real Apic research references** — Constitutional AI, Responsible Scaling Policy, interpretability/circuits work, Claude's Acceptable Use Policy. Generic "passion for AI" is a Lean No.
4. **Real Indian BFSI context** — RBI cloud/outsourcing guidelines, DPDPA 2023, data residency, Bedrock & Vertex availability in Mumbai/Singapore.
5. **Loop structure from the interview articles** — recruiter → HM → onsite Loop 1 (technical) → onsite Loop 2 (values + 20-min project deep dive + 40-min skeptical Q&A) → references.
6. **Two strategic risks the fit analysis flagged**: IC lateral-move from team leadership; performative vs. operationalized mission alignment.
7. **Sequence the Claude API artifact build into Module 4 CodeLabs** — the GitHub artifact that must exist before applying.
8. **Brutally honest grading** captured side-by-side: your answer + upgraded version + what the upgrade fixed.
9. **Running BFSI case study** threading every module — support agent assist, internal SOP search, RM copilot, compliance review, Claude Code dev productivity, executive analytics.

---

## Repo structure (target)

```
course-repository/
├── .gitignore
├── CNAME                                    # learnapic.sunmintz.com (deferred publish)
├── LICENSE
├── README.md                                # private-prep notice + curriculum table
├── mkdocs.yml                               # mirrors sun-course-genai
├── vibes/                                   # source materials + handoff (this folder)
│   ├── plan.md                              # this file
│   ├── CLAUDE.md                            # repo-context for future agents
│   └── handoff.md                           # session handoff
│   # (the 4 primary source files — meta-prompt, JD, fit-analysis PDF, current
│   #  resume — are kept locally only and not committed)
└── docs/
    ├── index.md                             # course landing — 12-module syllabus
    ├── overrides/                           # copied from genai
    ├── stylesheets/extra.css                # copied from genai + .personal admonition style
    ├── 01-Apic-Calibration/
    │   ├── INDEX.md
    │   ├── Notes/                           # 5 files
    │   ├── Drills/                          # 3 files (TMAY, Why Apic, Architecture Story)
    │   ├── Case-Study/                      # BFSI intro
    │   └── Interview-QA-Bank.md
    ├── 02-Personal-Narrative-and-Role-Fit/
    ├── 03-Customer-Discovery-and-Pre-Sales/
    ├── 04-Claude-Platform-Architecture/
    │   └── CodeLabs/                        # the GitHub artifact build sequence
    ├── 05-Solution-Architecture-Patterns/
    │   └── SystemDesigns/                   # BFSI HLDs/LLDs
    ├── 06-Evaluation-Architecture/
    ├── 07-RAG-in-Regulated-Enterprises/
    ├── 08-Agentic-and-Tool-Use-Architecture/
    ├── 09-Security-Privacy-Governance/
    ├── 10-Cost-Latency-Reliability/
    ├── 11-Storytelling-Workshops-Demos/
    ├── 12-Full-Loop-Simulation/
    ├── Drill-Tracker.md                     # central log of graded drills
    └── Interview-Questions/                 # cross-module Q&A aggregator
```

Modules 02–12 get folder + placeholder `INDEX.md` today; full content lands incrementally.

---

## Per-module template

Each module's `Notes/` covers the meta-prompt's 12 sub-sections, condensed into 4–6 markdown files:

1. What this module tests + concept framing
2. What good vs. weak answers look like
3. Architecture / Claude-specific depth
4. Safety & governance lens
5. Strong-Hire rubric
6. Module-specific BFSI case-study chapter

Plus: `Drills/` (graded practice), `Interview-QA-Bank.md` (~26 questions), `CodeLabs/` where relevant (Module 4), `SystemDesigns/` where relevant (Modules 5, 7, 8).

---

## Today's session — concrete deliverables

### A0. Pre-scaffold artifacts (FIRST — into `vibes/`)

- `vibes/plan.md` — verbatim copy of this plan
- `vibes/CLAUDE.md` — repo-context file for any future Claude Code session
- `vibes/handoff.md` — human-readable session handoff

### A. Scaffolding

- `mkdocs.yml`, `CNAME`, `README.md`, `.gitignore`, `LICENSE`
- `docs/overrides/`, `docs/stylesheets/extra.css` (copied)
- `docs/index.md` — 12-module syllabus landing
- All 12 module folders with placeholder `INDEX.md`

### B. Module 01 full content

- `INDEX.md` + 5 Notes + 3 Drills + Case-Study/01-BFSI-Customer-Intro + Interview-QA-Bank

### C. Run the three practice drills

After Module 01 lands, you give your real answers to:
1. 2-minute "Tell me about yourself"
2. 90-second "Why Apic"
3. One customer AI architecture story (your pick)

I grade on Strong Hire / Hire / Lean No / Strong No, push back hard, log results into `docs/Drill-Tracker.md`.

### D. End-of-session commit

Concrete next-action commitment: which drill to redo, whether to start Module 4's first CodeLab, or move to Module 02.

---

## Subsequent sessions (target sequence)

| Week | Focus | Output |
|---|---|---|
| 1 | Modules 1–2 + Apic-tailored resume `.docx` | Resume ready; drills polished |
| 2 | Modules 3–4 + Claude API artifact build | Working GitHub repo with SDK calls, tool use, structured output, eval harness |
| 3 | Modules 5–7 (architecture, evals, RAG) | BFSI HLD/LLD, eval framework, CISO objection responses |
| 4 | Modules 8–10 (agents, security, production) | Workshop deck draft; executive pitch (3 versions) |
| 5 | Module 11 (storytelling, demos, objections) | 5-min demo; objection-handling library |
| 6 | Module 12 (full loop simulation) | Mock recruiter, HM, technical, values rounds — all graded |

**Application trigger** (you don't apply until ALL four are true):

- [ ] Resume rewritten and reviewed
- [ ] Claude API artifact on GitHub
- [ ] TMAY + Why Apic at Strong Hire
- [ ] Clean lateral-move answer

---

## Verification (today's session)

1. `mkdocs serve` from `course-repository/` shows the full 12-module nav with Module 01 fully populated
2. Module 01 reads as calibrated to *you* — not generic SA prep
3. Three drill answers logged with brutally-honest grades and concrete upgrade suggestions
4. You walk away knowing the exact next session's deliverable

If you score Strong Hire on all three drills first attempt, the rubric is broken — not you. Expect at least one Lean No.
