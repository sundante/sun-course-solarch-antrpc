# 03 · Drill 01 — Discovery Call Simulation

> **Status: Outline.** Body fills in Week 2. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this drill is.** A 30-minute simulated discovery call with the running BFSI customer, run with a partner playing the customer role. Graded on the speak-time ratio, the question taxonomy deployment, and whether the underlying business problem surfaces.

> **What this drill is NOT.** A demo dry-run. Not a pitch rehearsal.

---

## Prompt

> *"You're 3 minutes into a 30-minute discovery call with the BFSI customer's CIO + CDO + CISO. The AE has just handed off to you. The CIO opens with: 'We want Claude to handle 60% of our internal employee support tickets.' Take it from there."*

---

## Time box

- **Run:** 27 minutes (after the 3-min architect intro).
- **Partner role:** plays CIO/CDO/CISO; should resist your questions occasionally, surface a constraint mid-call, and ask one off-script question.
- **Recording:** mandatory; review with timestamps.

---

## Rubric

### Strong Hire
- **Speak-time ratio:** architect ≤30% of call.
- **Question classes:** all 3 (context / problem-surface / constraint) deployed in correct order.
- **Real problem surfaces:** customer says the underlying business problem (not the requested AI feature) by minute 18.
- **Constraints enumerated:** at least 4 of {residency, integration, talent, timeline, regulatory, budget}.
- **No premature solutioning:** no architecture proposed before minute 24.
- **Close:** "What should I have asked you?" run cleanly.
- **Apic-frame:** safety/governance referenced once, in passing, in correct context.

### Hire
- Speak-time ratio 30–40%.
- 2 of 3 question classes deployed.
- Real problem surfaces only after the partner volunteers it.
- 3 of 6 constraints enumerated.
- Architecture proposed at minute 20–23 (slightly early but recoverable).

### Lean No
- Speak-time ratio >40%.
- Architecture proposed before minute 18.
- Customer's stated request taken at face value (chatbot = chatbot, not enterprise search).
- No constraint enumeration phase.

### Strong No
- Speak-time ratio >50%.
- Pitch slides reached for.
- "Apic's mission is…" appears.
- Use case accepted without questioning whether Claude is right.

→ Master rubric: [Drill Tracker](../../Drill-Tracker.md).

---

## Partner instructions (for the person playing customer)

Hand this to your drill partner before starting:

```
You are the CIO. The CDO and CISO are co-host on the call.
Your stated request: "We want Claude to handle 60% of our internal employee
support tickets."

The actual underlying problem (don't volunteer this — make the architect
surface it):
- Internal employees can't find policy documents fast enough.
- HR + IT helpdesk is overwhelmed by repetitive policy questions.
- A previous chatbot POC by another vendor hallucinated and got escalated.

Constraints (mention each only when probed):
- Data residency: Mumbai-only (RBI guidance).
- Integration: ServiceNow + Confluence + SharePoint.
- Sign-off: CISO has effective veto.
- Timeline: pressure for a "show value" demo in 4 weeks.

Behaviors:
- Once during the call, push back on a question with: "Can you tell us
  what Claude can do first?"
- Once during the call, ask: "How is this different from what we tried last time?"
- At minute 25, if architect hasn't surfaced the previous-vendor failure,
  volunteer it.
```

---

## Worked Strong Hire example (placeholder)

> *Body to be filled in Week 2 with annotated transcript timestamps and rubric scoring.*

---

## Common Lean No traps

### Trap 1 — Accepting the chatbot framing
Architect proposes a chatbot architecture without surfacing that the underlying problem is enterprise search.

### Trap 2 — Out-talking the customer
By minute 12 the architect has spoken more than the room. Ratio failed.

### Trap 3 — Premature competitor comparison
Customer mentions previous vendor; architect responds with "yes, GPT-4 has those issues, Claude is better." Strong No — sounds like sales.

### Trap 4 — Pitching the platform
"Let me tell you about Claude Opus 4.7…" before discovery is done. Lean No.

### Trap 5 — Skipping CISO
CISO is on the call but architect addresses only the CIO. The CISO ends call disengaged. Lean No.

### Trap 6 — Missing the time-pressure signal
Customer says "4-week demo." Architect doesn't push back on scope; accepts the timeline. Lean No.

---

## How to run this drill

1. Brief the partner with the script above.
2. Time the call. Mark architect-speak vs customer-speak with a stopwatch (or post-hoc from the recording).
3. After the call, score against the rubric.
4. Identify the *one* most consequential trap and rewrite that move as a written sentence.
5. Re-run with a different partner playing a different role mix (e.g. CISO-led).
6. Application trigger: 2 Strong Hires across 2 sessions with different partners.

---

## Cross-references
- Note: [01 — Discovery Call Structure](../Notes/01-Discovery-Call-Structure.md).
- Note: [03 — Stakeholder Mapping](../Notes/03-Stakeholder-Mapping.md).
- Sibling drills: [02 — Surface the Real Problem](02-Surface-the-Real-Problem.md), [03 — Three Discovery Objections](03-Three-Discovery-Objections.md).
- Case study: [BFSI Discovery Script](../Case-Study/01-BFSI-Discovery-Script.md).
- [Drill Tracker](../../Drill-Tracker.md).

## Strong-Hire bar for this drill
- ≤30% architect speak-time.
- All 3 question classes deployed in order.
- Real problem surfaced without partner volunteering.
- 4+ constraints enumerated.
- "What should I have asked you?" close runs.
