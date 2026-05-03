# 04 — Claude Platform Architecture

**Overview:** Know Claude's API surface, model selection trade-offs, cost optimization levers, and deployment options. This is table stakes: you must be able to recommend the right model for the right use case without hesitation.

---

## What This Module Tests

1. **Model selection judgment** — Can you recommend Opus 4.7 vs Sonnet 4.6 vs Haiku 4.5 given constraints (cost, latency, quality)?
2. **Platform knowledge** — Do you understand prompt caching, token economy, structured outputs, tool use?
3. **Deployment clarity** — Bedrock (AWS) vs Vertex (Google) vs first-party API? You should know the customer implications.

---

## Learning Path

**Recommended order:**

| Step | File | Purpose | Time |
|------|------|---------|------|
| 1 | [01-Claude-API-Surface](Notes/01-Claude-API-Surface.md) | API endpoints, message format, parameters | 15 min |
| 2 | [02-Model-Selection-Matrix](Notes/02-Model-Selection-Matrix.md) | Opus 4.7 vs Sonnet 4.6 vs Haiku 4.5: cost, latency, quality | 15 min |
| 3 | [03-Prompt-Caching-Architecture](Notes/03-Prompt-Caching-Architecture.md) | How prompt caching works, ROI, gotchas | 15 min |
| 4 | [04-Tool-Use-and-Structured-Outputs](Notes/04-Tool-Use-and-Structured-Outputs.md) | Agent patterns, function calling, JSON output | 15 min |
| 5 | [05-Deployment-Surfaces](Notes/05-Deployment-Surfaces.md) | First-party, Bedrock (AWS), Vertex (Google): tradeoffs | 15 min |

**Total notes time:** ~75 minutes

---

## Practice Exercises

| Drill | Focus | Time | Rubric |
|------|-------|------|--------|
| [01-30-Second-Model-Selection](Drills/01-30-Second-Model-Selection.md) | Customer says: "We need an LLM for customer support." Which model? Why? | 2 min | Reasoning, tradeoffs, business awareness |
| [02-Whiteboard-Prompt-Cache-ROI](Drills/02-Whiteboard-Prompt-Cache-ROI.md) | Draw out the ROI on prompt caching for a BFSI doc search use case | 10 min | Math, assumptions, customer benefit |
| [03-Bedrock-vs-Vertex-vs-First-Party](Drills/03-Bedrock-vs-Vertex-vs-First-Party.md) | Customer is on AWS. Should they use Bedrock or first-party API? | 5 min | Governance, data residency, cost, latency |

**Goal:** All 3 at **Hire** level minimum. **Strong Hire** means you can hold the line on what Claude can't do.

---

## Code Labs

Hands-on API practice:

- [01-First-Claude-Call](CodeLabs/01-First-Claude-Call.md) — Your first API call
- [02-Tool-Use](CodeLabs/02-Tool-Use.md) — Function calling and tool definitions
- [03-Structured-Output](CodeLabs/03-Structured-Output.md) — JSON schema enforcement
- [04-Prompt-Caching](CodeLabs/04-Prompt-Caching.md) — Cache implementation and ROI measurement
- [05-Custom-Eval-Harness](CodeLabs/05-Custom-Eval-Harness.md) — Build evaluation framework for your customer's use case

---

## Running BFSI Customer

- **[01-Claude-Platform-Decisions-for-BFSI](Case-Study/01-Claude-Platform-Decisions-for-BFSI.md)** — What model, features, and deployment surface for each BFSI use case?

---

## Knowledge Check

**Interview-QA-Bank** covers:
- Model selection & trade-offs (technical questions)
- Cost optimization (HM questions)
- Deployment strategy (executive questions)

👉 **Skim the Q&A Bank first.** Mark your 5 weakest questions.

**[Interview-QA-Bank](Interview-QA-Bank.md)**

---

## How to Work This Module

1. ✅ **Read all 5 Notes in order** (~75 minutes)
2. ✅ **Run the 3 Drills** — Time yourself; answers should be immediate and confident
3. ✅ **Run the 5 CodeLabs** — Get the Claude API in your hands; experiment
4. ✅ **Review the case study** — Apply your knowledge to real BFSI scenarios
5. ✅ **Skim Interview-QA-Bank** — Mark your 5 weakest questions
6. ✅ **Mark complete** when all Drills reach **Hire** level

**Strong-Hire bar:** You can explain token economy math and prompt caching ROI in customer language.

---

## What's Next

After this module:
- You can recommend Claude confidently in any conversation
- You understand cost and latency trade-offs
- You've written code that calls Claude
- You know deployment strategy for Indian BFSI (Bedrock India, Vertex India, data residency)

**Next Module:** [05-Solution-Architecture-Patterns](../05-Solution-Architecture-Patterns/INDEX.md)

---

## Quick Reference

**Key Files:**
- Notes: [Notes/](Notes/) (5 files)
- Drills: [Drills/](Drills/) (3 exercises)
- CodeLabs: [CodeLabs/](CodeLabs/) (5 implementations)
- Case Study: [Case-Study/](Case-Study/)
- Interview Q&A: [Interview-QA-Bank](Interview-QA-Bank.md)
