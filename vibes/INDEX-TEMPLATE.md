# INDEX.md Template for Modules

Use this template when creating or updating module INDEX.md files:

```markdown
# NN — Module-Name

**Overview:** One-liner describing what this module covers and why it matters for the interview.

---

## What This Module Tests

2-3 sentences on the key competencies or knowledge this module assesses.

---

## Learning Path

**Recommended order:**

| Step | File | Purpose | Time |
|------|------|---------|------|
| 1 | [01-Topic-Name](Notes/01-Topic-Name.md) | Description | X min |
| 2 | [02-Topic-Name](Notes/02-Topic-Name.md) | Description | X min |
| ... | ... | ... | ... |
| N | [N-Topic-Name](Notes/N-Topic-Name.md) | Final concept | X min |

**Total notes time:** ~Y minutes

---

## Practice Exercises

| Drill | Focus | Time | Rubric |
|-------|-------|------|--------|
| [01-Exercise](Drills/01-Exercise.md) | What's tested | X min | How to evaluate |
| [02-Exercise](Drills/02-Exercise.md) | What's tested | X min | How to evaluate |
| [03-Exercise](Drills/03-Exercise.md) | What's tested | X min | How to evaluate |

**Goal:** All at **Hire** level minimum. **Strong Hire** preferred.

---

## Running BFSI Customer

- **[01-Scenario-Title](Case-Study/01-Scenario-Title.md)** — Scenario description

---

## Knowledge Check

**N Interview Questions** across M categories:
- Category 1 (X questions)
- Category 2 (X questions)
- ...

👉 **Skim the Q&A Bank first.** Identify your 5 weakest questions.

**[Interview-QA-Bank](Interview-QA-Bank.md)**

---

## Code Labs (if applicable)

Hands-on implementations:

- [Lab Title](CodeLabs/lab-name.ipynb)
- [Lab Title](CodeLabs/lab-name.ipynb)

---

## System Designs (if applicable)

Production architecture patterns:

- [Design Title](SystemDesigns/design-name.md)
- [Design Title](SystemDesigns/design-name.md)

---

## How to Work This Module

1. ✅ **Read all Notes in order** (~Y minutes)
2. ✅ **Run the Drills with a timer** — raw answer first, then rubric
3. ✅ **Review the case study** — understand the scenario for this module
4. ✅ **Skim Interview-QA-Bank** — mark your weakest 5 questions
5. ✅ **Mark complete** when all Drills reach **Hire** level

---

## What's Next

Brief 1-2 sentence summary of where this fits in the overall curriculum.

**Next Module:** [NN-Module-Name](../NN-Module-Name/INDEX.md)

---

## Quick Reference

**Key Files:**
- Notes: [Notes/](Notes/) (N files)
- Drills: [Drills/](Drills/) (3 exercises)
- Case Study: [Case-Study/](Case-Study/)
- Interview Q&A: [Interview-QA-Bank](Interview-QA-Bank.md)
```

## Guidelines

1. **One-liner overview** should capture the "why this matters" for the interview
2. **Learning path table** should show realistic time estimates (usually 10-20 min per note)
3. **Practice exercises** should be timed and specific about what's being evaluated
4. **BFSI Customer** should thread through all modules for continuity
5. **Knowledge check** should always point to Interview-QA-Bank
6. **Quick reference** at bottom makes navigation easy
7. **Navigation arrows** at bottom link to previous/next module

## Example Time Budgets

- Light module (5 notes): 60-90 minutes total
- Standard module (5 notes + drills + Q&A): 90-120 minutes
- Heavy module (7-8 notes + labs): 150-180 minutes

## Consistency Checks

✅ All module INDEX.md files follow this structure
✅ Time estimates are realistic and tested
✅ Drill descriptions are specific (not generic)
✅ BFSI case study is described clearly
✅ Navigation links work (test before committing)
✅ File paths are relative and correct
