# 03 · Note 05 — Working with Account Executives

> **Status: Outline.** Body fills in Week 2. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** The architect-AE partnership model — when the AE leads, when the architect leads, how to honor customer urgency without being co-opted into sales escalation.

> **What this file is NOT.** A sales-ops manual. Not commission politics. Not "how to manage your AE."

---

## Why this matters

In Apic's pre-sales motion, the Applied AI Architect and the Account Executive are *peers*, not boss-and-report. The architect is the technical voice the customer trusts; the AE owns the deal mechanics. The partnership has predictable failure modes if either side blurs the line.

### What the AE owns
- The relationship cadence, calendar, executive coverage.
- Commercial structure (commit, pricing tier, contract shape).
- Internal Apic stakeholder management.
- "Is this a deal?" qualification.

### What the architect owns
- The customer's technical trust.
- Architecture, evals, deployment recommendations.
- "Is this technically the right shape?" judgment.
- "Should this be Claude at all?" judgment.

### What's shared
- The discovery-call structure.
- The customer's success criteria.
- The decision on when to bring engineering in.

---

## When the AE leads

- First-meeting orchestration. The AE has more context on the room dynamics than you will on day one.
- Pricing/commercial conversations. The architect should not improvise on commercial terms.
- Account-management cadence. Cadence is the AE's lever, not yours.
- Internal Apic forecasting. Don't speak to forecast.

## When the architect leads

- Technical discovery (the 25-min middle of call 1).
- Architecture review meetings.
- CISO-readiness reviews.
- Eval design workshops.
- Engineering-side escalations.
- "Should we walk away from this use case?" judgment calls.

## When you co-lead (and need to be calibrated)

- The "next steps" hand-off at end of call 1. Architect proposes shape; AE proposes timing.
- Customer-side escalations. Architect signals technical risk; AE signals commercial risk.
- POC scoping — architect on tech, AE on commercial fit.

---

## The escalation-without-being-co-opted move

A regular failure mode: the customer is pushing for a fast commit; the AE wants to honor it; the architect knows the use case isn't ready. Resolution:

> *"I'm with the AE on honoring the urgency. The shape that lets us honor it without shipping something that comes back is X — eval-gated, narrower scope, three-week sprint. The shape that doesn't let us honor it is Y — full POC, no eval, six-week timeline. Pick X."*

Architect signals technical risk in the language of *shape*, not *no*. The AE can sell the shape. The customer hears commitment, not refusal.

## When to flag a deal you're worried about

If the use case has a Pattern-4 "shouldn't be automated" smell (see [Note 04](04-When-to-Say-Claude-Is-Not-The-Tool.md)):

- Talk to the AE *before* the next customer call, not during.
- Bring a re-scope option, not a veto.
- Frame it as Apic's bar, not your personal preference.

## When to push back on AE pressure

The AE will sometimes ask for more architectural commitment than is honest. The architect's response:

> *"I can commit to X, which is what I can support technically. I can't commit to Y on this call without designing the eval first. If Y matters for the deal, let's bring engineering in for call N+1."*

This is *not* obstruction. It's calibration.

---

## Anti-patterns

### Anti-pattern 1 — Architect-as-AE
Architect starts answering pricing questions, hinting at discounts, scoping commercials. Strong No. The architect's currency is technical trust — spending it on commercials devalues both.

### Anti-pattern 2 — Architect-vs-AE
Architect treats the AE as a sales adversary to be managed. Strong No. The AE knows things you don't (calendar dynamics, decision-maker readiness, competitive context).

### Anti-pattern 3 — Silent-on-risk architect
Architect knows the use case is wrong but doesn't say so to the AE because "the deal is closing." Strong No. Apic's bar is the architect's bar.

### Anti-pattern 4 — AE replacement
Architect tries to replicate AE motions (commercials, exec coverage). Strong No. Different lever; different trust.

---

## What an AE wants from you in 5 lines

- Honest read on technical fit.
- Architecture-shaped re-scope options when things don't fit.
- Early flag on risks (CISO veto, residency, capability gap).
- Customer technical trust earned and maintained.
- No surprises in commercial conversations.

---

## Cross-references
- Sibling: [Note 03 — Stakeholder Mapping](03-Stakeholder-Mapping.md).
- Sibling: [Note 04 — When Claude Is Not the Tool](04-When-to-Say-Claude-Is-Not-The-Tool.md).
- Module 11: [Demo Discipline](../../11-Storytelling-Workshops-Demos/Notes/04-Demo-Discipline.md) — joint customer events with AEs.

## Strong-Hire bar for this file
- "Architect-AE peers, not boss-report" is reflex.
- Escalation-without-co-option move runs cleanly.
- "Apic's bar is my bar" used as a frame, not a defensive shield.
- Anti-patterns 1–4 recognizable in self-review.
