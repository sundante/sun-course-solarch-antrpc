# 03 · Drill 03 — Three Discovery Objections

> **Status: Outline.** Body fills in Week 2. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this drill is.** The three most common discovery-stage objections — cost, safety, lock-in — each handled in 90 sec at Strong-Hire grade. The objection responses you internalize for live calls.

> **What this drill is NOT.** A full objection-handling library (that's [Module 11 Note 05](../../11-Storytelling-Workshops-Demos/Notes/05-Objection-Handling-Library.md)). Not pricing rebuttals.

---

## The three objections

### Objection 1 — Cost
> *"Frontier models are too expensive for our volume. We can't run this at production scale."*

### Objection 2 — Safety / hallucination
> *"Last year a vendor's model hallucinated a regulatory fact in a CISO demo. How do we know Claude won't?"*

### Objection 3 — Lock-in
> *"We don't want to be locked into one model provider. What's our path if we want to switch?"*

---

## Time box

- **Per objection:** 90 sec response.
- **Total drill:** ~5 min including reading prompts cold.
- Cold variant: partner picks one of the three at random and adds a follow-up.

---

## Rubric

### Strong Hire
For each objection:
- **Acknowledge the legitimate concern** in 1 sentence (not "great question," but "yes, this is the right concern to raise").
- **Reframe** with a specific architectural answer, not a sales rebuttal.
- **Anchor with a concrete pattern** — model selection matrix, eval framework, multi-cloud deployment surface.
- **Close with a tradeoff acknowledgment** — what you're *not* claiming.
- 75–90 sec wall clock.

### Hire
- Acknowledge + reframe land, anchor is generic ("we have a model selection matrix") not specific.
- Trade-off acknowledgment missing.

### Lean No
- Sales-rebuttal shape ("Claude is actually cheaper than you think").
- Mission paraphrase ("Apic is committed to safety").
- Defensiveness audible.

### Strong No
- Over-claim ("Claude doesn't hallucinate") — Strong No on factual grounds.
- Dodging ("let's circle back") — Strong No.

---

## The Strong-Hire moves per objection

### Objection 1 — Cost
- **Acknowledge:** "Frontier-model cost is the architectural concern most often skipped early."
- **Reframe:** Model selection matrix — Opus for orchestration / Sonnet for the bulk / Haiku for high-volume classify. Plus prompt caching: long system prompts get cached, hit-rate optimization changes the cost shape.
- **Anchor:** *"In your support-agent assist case, the right shape is Sonnet 4.6 for the agent loop, Haiku 4.5 for the triage classifier, with shared cache for the policy context. That changes the cost-per-decision math by ~70% vs naive Opus."*
- **Trade-off acknowledgment:** *"This isn't free — you'll need an eval framework that accepts model substitution. We'll design that with you in stage 2."*
- → Module 04: [Model Selection Matrix](../../04-Claude-Platform-Architecture/Notes/02-Model-Selection-Matrix.md), [Prompt Caching Architecture](../../04-Claude-Platform-Architecture/Notes/03-Prompt-Caching-Architecture.md).

### Objection 2 — Safety / hallucination
- **Acknowledge:** "Yes — this is the concern your CISO will raise too. Claude hallucinates. Every frontier model does."
- **Reframe:** The architecture decides whether hallucination matters, not the model. Three controls: retrieval design (citation-grounded answers), refusal gates (when the model isn't confident), human-in-the-loop on customer-impacting actions. Plus eval-gated rollout (shadow → assist → autonomous).
- **Anchor:** *"For your compliance-doc-review case, we'd design the eval to score faithfulness against the source document, gate at 99%+ before going to assist mode, and never run it autonomous. Hallucination isn't 'made it go away'; it's 'made the failure mode caught and reversible.'"*
- **Trade-off acknowledgment:** *"We're not claiming zero hallucination. We're claiming bounded blast radius."*
- → Module 06, Module 07, Module 09.

### Objection 3 — Lock-in
- **Acknowledge:** "Right concern. Ten years of cloud history says lock-in matters."
- **Reframe:** Three layers of substitutability. (1) Deployment surface — Bedrock, Vertex, first-party. You can move between them with the same Claude. (2) Eval framework — model-agnostic. Same eval runs against any candidate model. (3) Tool schema and orchestration logic — written in your code, not Apic's.
- **Anchor:** *"In your architecture, we'd design the eval framework as a separate component that scores any candidate model against your acceptance threshold. If a non-Claude model crossed your threshold tomorrow, you could substitute it in under a sprint."*
- **Trade-off acknowledgment:** *"Substitutability isn't free — you pay a small architecture tax for the abstraction. The math works at your scale; might not at smaller scale."*
- → Module 04: [Deployment Surfaces](../../04-Claude-Platform-Architecture/Notes/05-Deployment-Surfaces.md). Module 06.

---

## Common Lean No traps

### Trap 1 — Sales rebuttal
"Claude is actually cheaper / safer / more open than you think." Reads as marketing. Lean No.

### Trap 2 — Over-claim
"Claude doesn't hallucinate at the bar your CISO needs." Strong No on factual grounds.

### Trap 3 — Dodging
"Let's set up a follow-up to discuss this." If the architect can't address it cold, the objection wins.

### Trap 4 — Mission cosplay
"Apic is deeply committed to..." Strong No.

### Trap 5 — Refusing the trade-off
"There are no trade-offs — this is just better." Strong No.

### Trap 6 — Premature competitor compare
"Yes, GPT-4 has those issues, Claude is better." Lean No — not what was asked.

---

## How to run this drill

1. Have a partner (or your phone recorder) pick one objection at random.
2. Speak the response cold, timer on, 90 sec.
3. Listen back. Score against the 4-step Strong-Hire structure.
4. For the criteria you missed, rewrite the response sentence.
5. Re-run with a different objection.
6. Application trigger: all 3 objections graded Strong Hire across 2 sessions.

---

## Cross-references
- Note: [04 — When Claude Is Not the Tool](../Notes/04-When-to-Say-Claude-Is-Not-The-Tool.md).
- Module 11: [Objection Handling Library](../../11-Storytelling-Workshops-Demos/Notes/05-Objection-Handling-Library.md) — the full 15-objection set.
- Module 04, Module 06, Module 09 — anchor depth.
- [Drill Tracker](../../Drill-Tracker.md).

## Strong-Hire bar for this drill
- 4-step structure (acknowledge / reframe / anchor / trade-off) deployed cleanly per objection.
- Anchors are *specific* (use case + model + control), not generic.
- Trade-off explicitly named, never hidden.
- All 3 objections at Strong Hire across 2 sessions.
