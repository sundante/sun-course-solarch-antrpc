# Interview Loop Map

> The actual stages of the Apic interview loop — synthesized from public interview write-ups (techprep.app, jobright.ai), the JD, and standard frontier-lab pre-sales loop structure. **Note: the public articles cover SWE / RS / MLE roles. The loop structure transfers; the technical-round content for an Applied AI Architect role is fundamentally different.**

---

## The full loop (estimated 4–6 weeks end-to-end)

```
Stage 1 — Recruiter screen          (30 min)
        ↓
Stage 2 — Hiring manager screen     (45–60 min)
        ↓
Stage 3 — Technical screen          (60–90 min — likely a deep-dive on past project + a structured customer architecture exercise)
        ↓
Stage 4 — Onsite Loop 1 (technical) (3 rounds × ~60 min — architecture whiteboarding, customer discovery sim, Claude platform deep-dive, eval design)
        ↓
Stage 5 — Onsite Loop 2 (values + project deep dive)  (2–3 rounds × ~60 min)
        ↓
Stage 6 — References + team matching
        ↓
Stage 7 — Offer
```

---

## Stage 1 — Recruiter screen (~30 min)

**What's tested:**

- High-level fit: do you understand the role accurately?
- Motivation: why Apic specifically? Why this role? Why now?
- Background: a fast walkthrough of your career arc
- Logistics: location, comp range, notice period, visa, willingness to travel

**What's a Lean No here:**

- Generic "passion for AI" with no specific Apic reference
- Misunderstanding the role (treating it as generic SA)
- Visible discomfort with the IC lateral move
- Inability to give a 2-minute "tell me about yourself" without rambling
- Not knowing what Claude for Work is (or only knowing the API)

**What's a Strong Hire here:**

- Tight 90-second to 2-minute career arc that lands at "and that's why this seat at Apic, now"
- Specific reference to one or two Apic things you've engaged with (Constitutional AI, RSP, Claude product feature)
- Confidence on the lateral move — owned, not apologized for
- Awareness of both Claude API and Claude for Work, with one sentence on each
- Smart question for the recruiter at the end (e.g., "what does the first 90 days look like for an Applied AI Architect joining this team?")

**Drills covered in Module 01:**
- [Tell Me About Yourself](../Drills/01-Tell-Me-About-Yourself.md)
- [Why Apic](../Drills/02-Why-Apic.md)

---

## Stage 2 — Hiring manager screen (~45–60 min)

**What's tested:**

- Past projects in real depth — you'll be asked to walk through one or two complex AI/architecture engagements
- Engineering judgment — trade-offs you accepted, decisions you made, what you'd do differently
- Customer-facing instinct — how you handle ambiguous customer asks, how you partner with sales, how you communicate up
- The lateral-move conversation in earnest

**What's a Lean No here:**

- A project walkthrough that's mostly outcomes ("we improved X by Y%") with no architecture depth, no failure modes, no trade-offs
- Inability to defend a past architectural choice when pushed ("why did you choose Neo4j over Weaviate?")
- Vague answers about how you'd partner with an account executive
- Defensiveness about the lateral move

**What's a Strong Hire here:**

- Project walkthrough structured as: (1) customer + business problem, (2) constraints, (3) architecture choices + trade-offs, (4) what went wrong, (5) what you'd change, (6) outcome
- Can defend past choices honestly, including admitting where you'd choose differently now
- Specific framing of how you operate in an AE partnership — "I own technical, AE owns commercial; I escalate when X, AE escalates when Y"
- Lateral move framed as direction, not regression — "depth over breadth at this stage"

**Drill covered in Module 01:**
- [Customer Architecture Story](../Drills/03-Customer-Architecture-Story.md)

---

## Stage 3 — Technical screen (~60–90 min)

For SWE / RS / MLE roles, this stage is the 90-min CodeSignal assessment. **For Applied AI Architect, the Apic-side technical screen will not be a CodeSignal coding test.** Based on the role profile, expect:

- **Past-project deep technical defense** (~30 min) — pick one project; the interviewer goes deep on the architecture; you defend choices
- **Live customer architecture exercise** (~30 min) — given a customer scenario (likely BFSI or similar regulated), design the Claude integration on a whiteboard / shared doc
- **Claude platform mini-quiz** (~10–15 min interleaved) — model selection, prompt caching, tool use, when to use which deployment surface

**What's a Lean No here:**

- Architecture without a discovery question first ("what does the customer actually need?")
- Skipping evals
- Skipping security boundaries / human-in-the-loop / audit
- Not knowing the Claude model lineup or making up details
- Choosing GPT or Gemini in the architecture (this is the Apic interview, not a model-agnostic interview)

**What's a Strong Hire here:**

- Opens with discovery questions before drawing any boxes
- Reasons through model selection explicitly (Opus 4.7 for complex reasoning, Sonnet 4.6 for the bulk, Haiku 4.5 for high-volume / latency-sensitive paths)
- Includes evals as a first-class architecture component, not an afterthought
- Names security, audit, and human-in-the-loop boundaries before being asked
- Can articulate the cost and latency model in rough numbers

**Modules that prep this stage:** 03 (discovery), 04 (Claude platform), 05 (architecture patterns), 06 (evals), 09 (security).

---

## Stage 4 — Onsite Loop 1 (technical, ~3 rounds × 60 min)

The technical onsite loop. Each round goes deeper into one capability:

### Round 1 — Architecture whiteboarding

A specific use case (probably one of: support agent assist, internal SOP search, RM copilot, compliance review). You design end-to-end on a whiteboard or shared doc. The interviewer asks "what if X" probes throughout — what if traffic 10x'd? what if the customer wants on-prem? what if Claude hallucinates a wrong number?

### Round 2 — Customer discovery simulation

The interviewer plays a CIO or CDO. You run a 30-minute discovery call. The probe: do you ask the right questions? Do you avoid solutioning before understanding? Do you know when to push back on the customer's stated request?

### Round 3 — Claude platform deep-dive + eval design

Open-ended technical Q&A on Claude (model selection, caching, tool use, structured output, deployment surfaces, MCP) plus design an eval framework for one specific use case.

**Modules that prep this loop:** 03 (discovery), 04 (platform), 05 (patterns), 06 (evals), 07 (RAG), 08 (agents).

---

## Stage 5 — Onsite Loop 2 (values + project deep dive, ~2–3 rounds)

**This is, per public intel, the most-feared and most-decisive part of the loop.** Behavioral and values rounds carry equal weight to technical rounds.

### Round 1 — Project deep dive (60 min total)

- 20-min presentation: pick *one* project, walk the panel through it
- 40-min "skeptical senior engineer" Q&A: pointed technical probes on every choice, every trade-off, every failure mode

What's tested: can you defend a real technical project for 40 minutes without breaking? Can you admit "I'd do this differently now" without it sounding like a confidence problem?

### Round 2 — Values / behavioral

The actual public examples (per techprep.app, jobright.ai):

- *"Tell me about a time you made a safety-first decision in a project, even if it meant a trade-off."*
- *"Tell me about a time when a technical misjudgment led to a project delay."*
- *"Describe a time you had a technical disagreement with a colleague."*
- *"What would you do if, midway through a project, you realized it was actually unfeasible?"*
- *"What do you think is the biggest risk of anthropomorphizing language models?"*
- *"Tell me about a time you did something that conflicted with your values."*
- *"How would you handle being assigned to a project you believe is unsafe?"*

Notice the pattern: every question probes whether you've internalized the tension between moving fast and being responsible. They want to hear the trade-off you accepted, not just that you valued safety.

### Round 3 (sometimes) — Executive communication round

You're asked to explain Claude's value (or a specific Claude architecture) to a hypothetical CEO / CIO / CISO. Multiple registers, sometimes in the same round.

**Modules that prep this loop:** 02 (narrative + lateral move), 09 (safety/governance), 11 (storytelling/registers), 12 (mock loop).

---

## Stage 6 — References + team matching

- References on past projects + customer-facing work (described as "thorough" in public write-ups)
- Team matching: the team you'd join is confirmed and finalized (which manager, which set of customers)

---

## Stage 7 — Offer

- Compensation + equity discussion
- Signing logistics, visa (if needed), start date

Apic is reportedly more flexible on negotiation than OpenAI's flat-comp model. Worth modeling against your alternatives.

---

## What's different for this role vs. SWE/RS/MLE loops

The public articles (techprep.app, jobright.ai) cover SWE / RS / MLE roles. **The differences for the Applied AI Architect role:**

| Aspect | SWE / RS / MLE | Applied AI Architect (this role) |
|---|---|---|
| 90-min CodeSignal | Yes (LRU cache, GPU scheduler, etc.) | No — the technical bar is architecture and discovery, not algorithmic coding |
| LLM-specific system design | Optional / role-dependent | Core — Claude-specific deployment patterns are central |
| Project deep dive | Yes | Yes — and **likely the highest-stakes round** |
| Values rounds | Equal weight | Equal weight — same |
| Customer discovery sim | Rare | Likely — this is core to the day-job |
| Live architecture whiteboard | Sometimes | Almost certainly |
| Coding artifact (take-home) | CodeSignal | The expected pre-application signal here is a **GitHub Claude artifact** (per the fit analysis) — not a take-home assigned by the loop |

**The implication:** the Apic interview articles are useful for *cultural calibration* (mission alignment, safety probing, project deep dive) but the *technical content* of your loop will be architecture/discovery/evals, not LRU cache and GPU scheduling. Don't grind LeetCode for this loop.

---

## Working memory for the loop

Three things to have at instant recall about the loop itself:

1. **The values/behavioral rounds are weighted equally with technical** — most candidates underprep these. You won't.
2. **The project deep dive is 20 + 40** — 20 minutes to present, 40 minutes to defend. Pick the project you can defend for 40 minutes; not the one with the best outcome metric.
3. **The Apic interview rewards specificity** — generic answers are Lean No, even if the underlying judgment is right. Lead with the specific example, then generalize, never the reverse.
