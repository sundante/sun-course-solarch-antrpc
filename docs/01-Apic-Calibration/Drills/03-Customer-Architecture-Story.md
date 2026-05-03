# Drill 03 — Customer Architecture Story

> The drill that the project deep-dive round is built on. 5 minutes of you presenting an architecture; in the real loop, 40 more minutes of skeptical Q&A. Pick the project you can defend for 40 minutes — not the one with the prettiest outcome metric.

---

## The prompt

> *"Walk me through a complex AI architecture you designed."*

Variants: *"Tell me about your most challenging AI engagement."* / *"Pick a recent project — walk me through the architecture."* / *"Describe a system you designed end-to-end."*

**Time budget:** 5 minutes for the standalone walkthrough. In the project deep-dive round, this becomes a 20-minute presentation followed by 40 minutes of pointed Q&A.

---

## Why this drill matters

This is *the* round of the loop, per public Apic interview intel. The project deep-dive in onsite Loop 2 is "the most-feared and most-decisive part of the loop." The interviewer becomes a skeptical senior engineer in a design review. They probe every choice. They are not adversarial — they are testing whether you can defend choices honestly under sustained scrutiny.

The 5-minute version (this drill) is the foundation. If you can't tell the story crisply in 5 minutes, you cannot defend it for 40 minutes. The structure has to be load-bearing.

---

## Project selection — the most important step

**Don't pick the project with the best outcome metric.** Pick the project where you can:

1. State the customer + business problem precisely (not "we improved CSAT")
2. Name the constraints that shaped the architecture
3. Defend at least three architectural choices, naming the alternative you rejected
4. Admit one failure mode honestly — what almost broke, what you caught in eval, what slipped to production
5. State what you'd change today, with reasoning
6. Surface the safety / governance lens unprompted
7. Translate the value to executive language

If you can do these seven things for a project, it's defensible for 40 minutes. If you can only do four of them, the round will go badly. Pick a project where you can do all seven.

!!! personal "Personal anchor"

    From your résumé, the candidate projects ranked by defensibility for the Apic loop:

    | Project | Defensibility | Why |
    |---|---|---|
    | **Agentic Hybrid GraphRAG (GenAI services firm)** | Highest | Real architecture choices (graph + vector + agentic), real trade-offs (Neo4j vs alternatives, sync vs async, RAGAs gating), production deployment, multi-million-dollar engagement. The interviewer will love it. |
    | **Healthcare Multi-Agent Platform (healthcare AI consultancy)** | High | HIPAA-aligned, recent, Claude-aware, governance-first by design. The strongest *Apic-narrative* fit — but slightly less mature in your delivery story than the GraphRAG work. |
    | **Voice AI on Vertex (GenAI services firm, 40% CX lift)** | Medium-high | Production-scale, GCP-native, agent-state across sessions. Outcome metric is sharp. The risk: Voice AI architecture is less Apic-fit (latency vs reasoning trade-offs read differently). |
    | **Equity-trading surveillance (global investment bank)** | Medium | Production AI in regulated FSI; great safety-as-throughput angle; older (2021–22) so the interviewer may probe how you'd architect it differently today. |

    **Recommended primary**: Agentic Hybrid GraphRAG — strongest architecture story.
    **Recommended backup if asked for a second project**: Healthcare Multi-Agent Platform — strongest Apic-narrative fit.
    **Save for the values round, not the architecture round**: equity-trading surveillance.

    Run the GraphRAG story end-to-end with this drill first. The Healthcare Multi-Agent one second.

---

## The Strong Hire structure (5-minute version)

Six beats, ~50 seconds each:

### Beat 1 — Customer + business problem (~50 sec)

Specifically: who the customer was (industry, scale, regulatory posture), what the actual business problem was (not the requested AI solution — the underlying problem), and why a previous approach hadn't worked.

Pattern:
> *"The customer was [industry, scale, regulatory posture]. Their underlying business problem was [specific], which they were currently solving by [current approach] — that approach had [specific limitation]. The ask we got was [requested AI solution] — but the discovery showed the real problem was [refined understanding]."*

### Beat 2 — Constraints that shaped the architecture (~50 sec)

Three to five hard constraints. Not "scalable" or "performant" — concrete: data residency, latency p95, RBAC at retrieval layer, hallucination tolerance, integration with named existing systems, compliance regime.

Pattern:
> *"The hard constraints were: [constraint 1, with the specific number or rule], [constraint 2], [constraint 3]. The constraints we negotiated were [softer ones]. The constraints we explicitly *de-scoped* were [things we said no to]."*

### Beat 3 — Architecture choices + trade-offs (~70 sec)

The longest beat. Walk through 3 major choices, each with the alternative you rejected and why.

Pattern:
> *"At the retrieval layer, we chose [X] over [alternative Y] because [reason — usually tied to a constraint from Beat 2]. At the orchestration layer, we chose [X] over [Y] because [reason]. At the eval layer, we chose [X] over [Y] because [reason]. The trade-off we accepted was [specific trade-off — e.g., 'higher latency in exchange for synchronous accuracy because the downstream workflow couldn't tolerate eventual consistency']."*

### Beat 4 — Failure mode + what you caught (~50 sec)

The honesty beat. One specific failure mode you encountered — caught in eval ideally, or in early production — and how the architecture responded.

Pattern:
> *"The thing that almost broke us was [specific failure mode]. We caught it because [specific eval / observability mechanism]. We responded by [specific architectural change]. The lesson was [what you'd take to the next project]."*

### Beat 5 — Safety / governance lens (~40 sec)

Surface this unprompted. The architectural choice that wasn't the most-discussed but was the most-important from a safety perspective.

Pattern:
> *"From a safety lens, the under-discussed but most-important architectural choice was [specific — e.g., RBAC at the retrieval layer, or the human-in-the-loop approval gate, or the refusal-pattern tuning]. The model can't [leak / act on / hallucinate about] what the architecture doesn't let it see / do."*

### Beat 6 — Executive value + what you'd change today (~40 sec)

Close with the business framing and the honest "I'd do differently now" beat.

Pattern:
> *"For the executive: the value was [measurable outcome] tied to [downstream business KPI]. What I'd change today: [specific architectural change], because [reason — usually 'the platform / model has matured since' or 'I underestimated [X]']."*

---

## What gets each grade

### Strong No

- Walks through a project chronologically ("first we built X, then we built Y")
- No customer detail, no constraints, no trade-offs
- Outcome-only ("we improved CSAT 40%")
- Cannot defend a choice when probed
- Pretends nothing went wrong

### Lean No

- Has a clear architecture but is structured as a feature list
- Names some technologies but can't defend the choices
- No failure mode admitted
- No safety lens surfaced
- No "I'd change this today" beat

### Hire

- Six-beat structure mostly intact
- Trade-offs named, choices defensible
- Failure mode admitted
- Missing one of: safety lens, executive translation, "I'd change" beat

### Strong Hire

- Six beats land in 5 minutes with steady cadence
- Each architectural choice is named with the alternative rejected
- Failure mode is honest and specific
- Safety lens is surfaced unprompted
- Executive value is translated cleanly
- "I'd change this today" is grounded in specific reasoning, not vague humility
- Can transition into the 40-minute Q&A by saying "happy to go deeper on any of those" without panic

---

## What the 40-min Q&A will probe

The probes are predictable. Have prepared positions:

1. **"Why X over Y at the [retrieval / orchestration / eval] layer?"** — name your reason; if pushed, name the trade-off; if pushed further, admit "the alternative would have worked too — here's why I chose what I did."
2. **"What if traffic 10x'd?"** — talk about which layer breaks first, what you'd horizontally scale, what you'd cache, where the cost ceiling lives.
3. **"What if the customer asked for on-prem?"** — talk about what changes architecturally (model deployment surface, data flow, audit logging, eval harness portability).
4. **"What's the worst-case hallucination scenario?"** — name the worst case honestly, then describe the mitigations you architected (refusal patterns, citation requirements, human-in-the-loop, eval gates).
5. **"How would you have evaluated this differently?"** — your eval framework is a first-class architecture artifact; defend it like one.
6. **"What was the commercial outcome?"** — translate to business KPI, contract size, expansion potential.
7. **"What surprised you?"** — have a real surprise ready. Saying "nothing surprised me" is a Lean No.

---

## Common Lean No traps

1. **Outcome-first.** Leading with "we improved CSAT 40%" instead of the business problem signals you're optimizing for impressing, not for clarity.
2. **Technology name-dropping.** "We used Neo4j, FAISS, LangChain, LangSmith, RAGAs, Vertex AI..." without choices defended is just a tool list.
3. **Pretending nothing went wrong.** A project that ran perfectly is either fictional or shallow. Admit a real failure mode.
4. **Skipping the safety lens.** The Apic interview will eventually ask. Surfacing it unprompted is the move.
5. **Missing the executive translation.** Without the business value beat, the story reads as engineering-internal.
6. **No "I'd change."** Saying "I wouldn't change anything" reads as low-self-awareness or rehearsed.

---

## Practice protocol

1. **Pick the project.** Per the personal note above, default to the GenAI services firm GraphRAG.
2. **Write the 5-minute version using the six beats.** Do not bullet-point — write it as you'd say it.
3. **Time it aloud.** Cut to ≤300 sec.
4. **Bullet-point the structure.** Six beats → talking points only.
5. **Speak from bullets.** Record.
6. **Have a colleague (or yourself in 24 hours) ask 3 of the 7 probable Q&A probes** above. Practice the responses.
7. **Iterate** until the 5-minute version is steady AND you have positions on at least 5 of the 7 probable probes.
8. **Compress to 90 seconds** for use in the recruiter or HM screen — the structure stays the same; the depth shrinks.

---

!!! personal "Personal anchor"

    ### Recommended choice: the GenAI services firm Agentic Hybrid GraphRAG

    **Why this one:** The strongest defensibility profile across all 7 of the criteria above. You have:

    - Real customer + business problem
    - Hard constraints (likely residency, RBAC, latency)
    - At least 3 defensible architectural choices (graph + vector hybrid; ReAct + self-reflection orchestration; RAGAs as gating)
    - At least one real failure mode (you mentioned RAGAs catching things in eval)
    - Production deployment with measurable outcome
    - Safety lens (RBAC at retrieval; eval as gate not metric)
    - Multi-million-dollar contract value as executive frame

    ### Six-beat skeleton for the GenAI services firm GraphRAG (fill in the customer-specific details)

    1. **Customer + business problem** — "The customer was [industry, regulatory posture]. Their underlying problem was [specific operational problem]. The current solution was [previous approach]; its limitation was [specific]."
    2. **Constraints** — "Hard constraints: data residency [specific region requirement], RBAC at retrieval layer (no bypass through query rewriting), p95 retrieval latency under [number] ms, audit trail per query."
    3. **Architecture choices** — "Hybrid retrieval: graph for [reason], vector for [reason], chosen because pure-vector missed [X] and pure-graph missed [Y]. Orchestration: ReAct + self-reflection over single-pass because [reason]. Eval: RAGAs Faithfulness as a *deployment gate*, not just a monitoring metric, because [reason]."
    4. **Failure mode** — "The thing that almost broke us was [specific — recommend recalling the actual one from the GenAI services firm]. Caught in eval at the Faithfulness threshold because we'd built it as a gate. Architectural fix: [specific change]."
    5. **Safety lens** — "From a safety lens, the under-discussed but most-important choice was RBAC at the retrieval layer — the LLM can't leak what the architecture doesn't let it see. The other was the eval-as-gate posture; production was contingent on a Faithfulness threshold being met."
    6. **Executive value + what you'd change** — "For the executive: [specific operational KPI improvement] tied to [downstream business outcome]. What I'd change today: I'd push harder for the customer to invest in the eval set up front; we underbuilt it because of timeline pressure. Today I'd treat the eval set as a contractual deliverable, not a delivery internal artifact."

    Replace the bracketed sections with the actual specifics from your engagement (you have the real numbers; this skeleton is the structure).

    ### What to watch for in the 40-min Q&A

    Three probes you should *expect* given this story:

    - **"Why hybrid? Couldn't pure-vector with better embeddings have worked?"** — Have a precise answer. If your answer is "graph captured relationships vector embeddings flatten" — ground it in a specific query type the customer ran.
    - **"How did you size the eval set?"** — Have a number and methodology. "We started with [N] examples curated by [SME role], grew to [M] over the engagement, balanced across [categories]."
    - **"What model did you use? Why not Claude?"** — Be honest. Whatever model you used, you can frame it as "the model selection was constrained by [customer choice / available platform], and the architecture was model-agnostic at the inference layer — moving to Claude today would be straightforward and would benefit from [specific Claude advantage]."

---

## Your turn

When you're ready, write your raw 5-minute architecture story below — pick a project, work through the six beats, write it as you'd speak it. Aim for ≤500 seconds spoken (about 1000–1100 words written).

I'll grade on the rubric above, write the upgraded version, anticipate the most-likely Q&A probes, and log the result in [Drill Tracker](../../Drill-Tracker.md).

```
[Your raw answer goes here]
```
