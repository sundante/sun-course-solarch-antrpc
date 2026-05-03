# 03 · Drill 02 — Surface the Real Problem

> **Status: Outline.** Body fills in Week 2. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this drill is.** Five customer requests, each containing a hidden real problem. Drill: in 90 sec each, ask 3 probing questions that get from the requested AI feature to the underlying business problem.

> **What this drill is NOT.** A solutioning drill. Don't propose architecture during this drill.

---

## Prompts (5 customer asks)

### Ask 1 — *"We want Claude to write our compliance reports."*
**Hidden real problem (don't peek):** Auditor finds inconsistencies between sections. Underlying need is structured drafting + consistency-checking with review gates, not free-form generation.

### Ask 2 — *"We want a chatbot for relationship managers to query customer data."*
**Hidden real problem:** RMs spend 30% of their time switching between 5 systems to assemble a customer 360. Underlying need is integration + UX, not a chat interface.

### Ask 3 — *"We want Claude to do contract review."*
**Hidden real problem:** Legal team has a 3-week backlog and the bottleneck is *triage* — which contracts need detailed review. Underlying need is classification + routing, not full review.

### Ask 4 — *"We want a Claude agent to handle vendor onboarding."*
**Hidden real problem:** Onboarding fails because 4 internal teams pass forms that contradict each other. Underlying need is process redesign, not automation; Claude can help draft, but the process problem is upstream.

### Ask 5 — *"We want Claude to summarize all our internal Slack channels."*
**Hidden real problem:** Leadership doesn't know what's happening across 12 product teams. Underlying need is structured weekly briefs from team leads, not Slack summarization (which would surface noise + leak signal).

---

## Time box

- **Per ask:** 90 sec to deliver 3 probing questions.
- **Total drill:** ~10 min including reading the asks cold.

---

## Rubric

### Strong Hire
For each ask:
- 3 questions delivered, each in a different class (context / problem-surface / constraint).
- At least one question that *refuses the surface framing* without being adversarial. *("If the AI piece weren't part of this, how would you solve this problem today?")*
- Identifies the real problem **before** offering any architecture.
- No competitor comparison anywhere.

### Hire
- 3 questions delivered but all in the same class (e.g. all "what" questions).
- Real problem surfaces but only because the prompt obviously hints at it.

### Lean No
- Architect immediately offers solution architecture (Lean No).
- Questions are pitch-shaped ("would Claude be useful for…?").
- Surface ask accepted as the requirement.

### Strong No
- Architect skips probing entirely and pitches Claude features.
- "Yes we can definitely do that" said without further investigation.

---

## The 3-question template

The Strong-Hire structure for any customer ask:

1. **Context question** — *"Walk me through what's happening today before any AI gets involved."*
2. **Problem-surfacing question** — *"What's the business outcome you're chasing? What would success look like in 6 months?"*
3. **Constraint or counter-factual question** — one of:
   - *"If the AI piece weren't part of this, how would you solve this problem?"*
   - *"What did the previous attempt teach you?"*
   - *"Who has to sign off before this goes live?"*

If all 3 questions land, the customer surfaces the real problem in their *own* words.

---

## Common Lean No traps

### Trap 1 — Solution-shaped questions
*"Would Claude help if it could…"* — that's a pitch disguised as a question. Lean No.

### Trap 2 — Single-class questioning
3 "what" questions in a row. The constraint and counter-factual classes never get used. Real problem doesn't surface.

### Trap 3 — Accepting the AI framing
Treating "we want Claude to X" as the requirement. The requirement is the *outcome*, not the AI shape.

### Trap 4 — Over-leading
*"It sounds like what you actually need is…"* — finishing the customer's thought for them. They'll agree to be polite, then disagree internally.

### Trap 5 — Skipping to constraints
Constraint questions before context questions. Sounds like an audit, not discovery.

---

## How to run this drill

1. Cover the "hidden real problem" lines.
2. Read each ask cold; deliver 3 probing questions in 90 sec.
3. Reveal the hidden problem after answering.
4. Score: did your questions, in their own logic, surface the hidden problem?
5. For asks where you missed, write the *one* question you should have asked.
6. Application trigger: 4 of 5 asks graded Strong Hire across 2 sessions.

---

## Cross-references
- Note: [01 — Discovery Call Structure](../Notes/01-Discovery-Call-Structure.md) — the question taxonomy.
- Sibling drill: [01 — Discovery Call Simulation](01-Discovery-Call-Simulation.md) — applies the same skill end-to-end.
- [Drill Tracker](../../Drill-Tracker.md).

## Strong-Hire bar for this drill
- 3 questions per ask, in 3 different classes, in 90 sec.
- Real problem surfaces in customer's own words.
- No solutioning during the drill.
- 4 of 5 Strong Hires across 2 sessions.
