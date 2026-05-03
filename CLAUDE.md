# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A private **MkDocs Material** interview-prep course for the **Apic Applied AI Architect** role (Mumbai, Pre-Sales IC, India/APAC). Module 01 (Apic Calibration) is fully written at Strong-Hire grade. Modules 02–12 have complete folder scaffolding (INDEX.md, Notes/, Drills/, Case-Study/ stubs with H2/H3 outlines) — body content is the Week 1–6 build task per `vibes/Build-Plan.md`. Mirrors the structure and theme of the public [`sun-course-genai`](https://github.com/sundante/sun-course-genai) course.

**This site is not deployed.** A `CNAME` exists for `learnapic.sunmintz.com` for future use, but no GitHub Pages activation. Generic content will eventually be merged into a separate `sun-course-solarch` (AI Solutions Architect) course — write for that future merge.

For session state and the content schedule, read in this order:
- [`vibes/handoff.md`](vibes/handoff.md) — current session state and next pickup point
- [`vibes/plan.md`](vibes/plan.md) — the approved 12-module plan
- [`vibes/Build-Plan.md`](vibes/Build-Plan.md) — 6-week file-by-file schedule with status flags

## Commands

```bash
# Local preview (with live reload)
pip install mkdocs-material
mkdocs serve              # → http://127.0.0.1:8000

# Validate links + nav before commit
mkdocs build --strict     # exits non-zero on broken links

# Build static site to ./site/ (gitignored)
mkdocs build
```

There are no tests, no linter, no application code. The "build" is the rendered docs site.

## Architecture

### Per-module template

Every module under `docs/##-Module-Name/` follows the same shape:

```
docs/01-Apic-Calibration/
├── INDEX.md                      # module overview + what-this-tests
├── Notes/                         # 4–6 concept notes
├── Drills/                        # graded practice (in !!! personal blocks)
├── Case-Study/                    # BFSI running-case chapter for this module
├── Interview-QA-Bank.md           # ~26 graded questions
├── CodeLabs/                      # Module 04 only — Claude API artifact build
└── SystemDesigns/                 # Modules 05, 07, 08 — BFSI HLDs/LLDs
```

`docs/Drill-Tracker.md` is the central log of all graded drill answers across modules. `docs/Interview-Questions/` is a cross-module Q&A aggregator — entries there *link to* the per-module `Interview-QA-Bank.md` (don't duplicate content).

### The merge-friendliness contract

This course will be merged into `sun-course-solarch` later. To keep the merge cheap:

- **Note bodies are written generically** — every concept note explains the topic in a way that works for any candidate prepping for any AI Solutions Architect role.
- **Personal anchors live in `!!! personal "Personal anchor"` admonitions.** Candidate-specific stories, lateral-move framing, graded drill answers all go in these blocks.
- **The merge is a grep-and-strip** — when porting, remove the `!!! personal` blocks and the generic body remains.

When writing a note, ask: "would this be the same advice for any Pre-Sales Architect candidate?" If yes, it's body. If no, it's a `!!! personal` block.

A second admonition class `!!! grade` is reserved for graded drill verdicts. Both are styled in [`docs/stylesheets/extra.css`](docs/stylesheets/extra.css).

### Theme

Material for MkDocs with `custom_dir: docs/overrides`, the solar-yellow palette from `sun-course-genai`, Inter + JetBrains Mono fonts, Mermaid + admonitions + tabbed + details + footnotes extensions enabled. **Do not change the palette without asking** — it is shared with `sun-course-genai` and the future `sun-course-solarch` course.

## Hard constraints

1. **Never publish.** No `mkdocs gh-deploy`. The CNAME exists but the domain is not active.
2. **Personal identifiers are already redacted** — full name and past-employer names were stripped from the initial commit. Do not re-introduce them when writing new content. Use the established analogues:
   - Past employers: *the healthcare AI consultancy*, *the GenAI services firm*, *the global investment bank*, *the energy-analytics platform*, *the US financial services firm*, *the venture builder*, *the global SI*
   - Author identity: *the candidate*
3. **Four source files are kept locally only and not committed**: the meta-prompt (`1-prompt.md`), the JD (`AppliedAIArchitect.md`), the fit-analysis PDF, and the current résumé. The `vibes/` folder contains only `plan.md`, `handoff.md`, `Build-Plan.md`. Do not write code that depends on the missing files being present.
4. **Real April-2026 Claude lineup** in any model-selection content: Opus 4.7 (`claude-opus-4-7`), Sonnet 4.6 (`claude-sonnet-4-6`), Haiku 4.5 (`claude-haiku-4-5-20251001`).
5. **Real Indian BFSI context** in case-study and architecture work: RBI guidelines, DPDPA 2023, in-India residency, Bedrock + Vertex India regions. Not American examples.

## Voice & content rules

- **Second-person personal-prep voice** ("you," "your"). Brutally honest.
- **Echo the JD verbatim** where possible: "trusted technical advisor," "paint the vision," "develop evals," "common integration patterns."
- **Real Apic research references**: Constitutional AI, Responsible Scaling Policy, interpretability/circuits work, Acceptable Use Policy. Generic "passion for AI" is a Lean No on mission alignment.
- **Real Indian BFSI context**: RBI guidelines, DPDPA 2023, data residency, Bedrock and Vertex India regions. Not American examples.
- **All drill answers are graded on the rubric below.** When a drill answer is Lean No, say "Lean No" — don't soften it.

## Grading rubric

| Grade | Criteria |
|---|---|
| **Strong Hire** | Specific, technically credible, mission-aligned without being performative, references actual Apic work, shows architecture depth + customer empathy + safety judgment, owns the lateral-move narrative confidently |
| **Hire** | Mostly right but missing one of: specificity, depth, or principal-level maturity. Fixable with one revision |
| **Lean No** | Generic, buzzword-heavy, mission alignment is performative or absent, architecture is shallow, lateral-move framing is defensive, or doesn't reference any specific Apic work |
| **Strong No** | Wrong on safety judgment, dishonest about gaps, oversells, factually incorrect about Claude/Apic, or signals misalignment with safety-first culture |

When grading: name the grade, list 1–3 specific things missing, then write the upgraded version.

## Running BFSI case study

A composite large Indian BFSI enterprise threads every module's `Case-Study/` chapter. Six use cases:

1. Customer support agent assist
2. Internal policy and SOP search
3. Relationship manager copilot
4. Compliance document review
5. Developer productivity with Claude Code
6. Executive analytics assistant

**Hard constraints:** RBI governance, in-India residency, RBAC at branch/region/role/department level, hallucination tolerance approaching zero, HITL before customer-impacting actions, Salesforce + ServiceNow + Genesys + SharePoint + Confluence + Snowflake + Databricks + Kafka + IAM/SSO integration. **The CISO is the gatekeeper, not a stakeholder** — design for that posture from day one.

Full customer profile: [`docs/01-Apic-Calibration/Case-Study/01-BFSI-Customer-Intro.md`](docs/01-Apic-Calibration/Case-Study/01-BFSI-Customer-Intro.md).

## Application trigger checklist

Do **not** apply until all four are true:

- [ ] Apic-tailored resume `.docx` written and reviewed
- [ ] Claude API artifact on GitHub (Python SDK + tool use + structured output + prompt caching + eval harness)
- [ ] "Tell me about yourself" + "Why Apic" both at **Strong Hire**
- [ ] Clean, non-defensive lateral-move answer (Module 02 output)

## How to pick up work in a new session

1. Read `vibes/handoff.md` for the latest state and next session pickup point.
2. Check `docs/Drill-Tracker.md` for graded drill history.
3. Open `vibes/Build-Plan.md` to find the current week's files and flip status flags as files are completed.
4. Ask: "What did we commit to as the next session's deliverable?" (logged at the end of `vibes/handoff.md`).
