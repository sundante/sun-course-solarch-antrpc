# What Good vs Weak Looks Like

> The grading rubric used across every drill in this course, with concrete examples calibrated to Apic's bar.

---

## The rubric

| Grade | Criteria | When you'd see this |
|---|---|---|
| **Strong Hire** | Specific, technically credible, mission-aligned without being performative, references actual Apic work, shows architecture depth + customer empathy + safety judgment, owns the lateral-move narrative confidently | Top 10–15% of candidates. The interviewer leaves wanting to fight for you. |
| **Hire** | Mostly right but missing one of: specificity, depth, principal-level maturity. Fixable with one revision. | Solid. Most offers come from this band — but for a role this competitive, the safer aim is Strong Hire on at least the headline questions. |
| **Lean No** | Generic, buzzword-heavy, mission alignment is performative or absent, architecture is shallow, lateral-move framing is defensive, or doesn't reference any specific Apic work | The most common failure mode for senior candidates who haven't prepped specifically. Recoverable in subsequent rounds, but each Lean No raises the bar on the next round. |
| **Strong No** | Wrong on safety judgment, dishonest about gaps, oversells, factually incorrect about Claude/Apic, or signals misalignment with safety-first culture | Loop-killer. Even one Strong No in a values round is usually disqualifying. |

---

## Worked example: "Why Apic?"

Same question. Four grades.

### Strong No

> "I think AI is going to change the world, and Apic is one of the leaders. I want to be part of that. The pay is great and the team is impressive. I'm a huge fan of Claude — it's better than ChatGPT in my opinion."

Why: factually thin (no specific Apic reference), motivation-shallow (pay + prestige), and contains a comparison ("better than ChatGPT") that's both unsubstantiated and tonally wrong for an Apic interview. The "I'm a fan" framing also reads as consumer, not architect.

### Lean No

> "I'm passionate about safe and beneficial AI. Apic's mission to create reliable, interpretable, and steerable AI systems aligns deeply with my own values. I've spent my career building AI responsibly, and I want to be part of a team that takes that seriously."

Why: this is a *paraphrase of Apic's mission statement read back to them*. It contains zero specifics. It claims values-alignment without evidence. It would land identically for any candidate who read the careers page. **This is the most common Lean No** — and it's a Lean No precisely because it's so common.

### Hire

> "Two things. First, I've been operating in regulated AI for the last few years — HIPAA-aligned PHI handling, SHAP-based explainability, governance-first delivery — and Apic is the lab where that operating mode is the design center, not a constraint to fight against. Second, I'm interested in the Indian BFSI adoption curve specifically, and Mumbai gives me direct access to the customers I think Claude should be central for. I want to be the architect those customers trust."

Why: this is *good*. Specific, evidence-backed, geographically credible. What's missing for a Strong Hire: no reference to specific Apic research or product, and no acknowledgement of why *this seat* (Pre-Sales IC) is the right shape for the trajectory.

### Strong Hire

> "Two things, plus a third I've thought about. First — operating-in-safety-mode is already how I work. Three years of HIPAA-aligned PHI handling at the healthcare AI consultancy, SHAP-based explainability shipped as a feature at the GenAI services firm, governance-first architecture as the default. Apic is the lab where that operating mode is the design center. Constitutional AI and the RSP are real artifacts to me, not branding — they shape what 'safe enough to deploy' means at the architectural layer.
> 
> Second — Indian BFSI specifically. Mumbai-based, regulated-by-RBI, data-residency-conscious, hallucination-tolerance approaching zero. These are the customers Claude is uniquely positioned for, and where the safety properties matter most commercially.
> 
> Third — and I've thought about this — this is an IC lateral from team leadership. The right framing is depth over breadth at this stage. Going deep with the strongest customers on the most important AI being deployed today is the lever I want to pull next, not adding more team members. The choice signals what I think the next two years should be."

Why: specific (HIPAA, SHAP, governance, RBI, Mumbai, Constitutional AI, RSP), credible (claims backed by real career evidence), self-aware (preempts the lateral-move risk before it's asked), and tonally Apic-fit (no performative passion, just operating-mode evidence).

---

## Worked example: "Walk me through a complex AI architecture you designed."

### Strong No

> "I led an AI platform that integrated GPT-4 with our customer support system. We achieved 40% improvement in customer satisfaction. The architecture used microservices, vector databases, and cloud infrastructure."

Why: vague, name-drops technologies without choices, has no constraints, no failure modes, no trade-offs, and uses GPT-4 (in an Apic interview). The 40% number floats with no methodology.

### Lean No

> "At the GenAI services firm we built an agentic Hybrid GraphRAG system for an enterprise customer. It used Neo4j, FAISS, LangChain, and LangSmith. We deployed on GCP with Vertex AI. The architecture had a retrieval orchestration layer, a graph query layer, and a vector retrieval layer. We measured it with RAGAs — Faithfulness, Relevance, Precision@K. It went to production and the customer was happy."

Why: this is *technically accurate* and from real experience, but it's structured as a feature list. There's no business problem, no constraints, no trade-offs, no decisions defended, no failure modes. It tells the interviewer *what you built* but not *why* or *what you learned*. A senior interviewer will probe and the candidate will need to scramble.

### Hire

> "I'll walk you through the agentic Hybrid GraphRAG we built at the GenAI services firm. The customer's problem was [specific problem]. They had constraints around [residency, latency, etc.]. We chose hybrid retrieval — graph + vector — because [specific reason]. The biggest trade-off was [X vs. Y, why we chose X]. The thing that almost killed us was [specific failure mode] — we caught it in eval before production. What I'd change now is [specific thing]."

Why: this is *Hire-grade*. Structured, defensible, includes a failure mode, includes a "I'd change this now." What's missing for Strong Hire: the safety/governance lens isn't surfaced (RBAC at retrieval, audit logging, hallucination tolerance, refusal patterns), and the executive value translation isn't there.

### Strong Hire

> "I'll walk you through the agentic Hybrid GraphRAG we built at the GenAI services firm. The customer was [specific], the business problem was [specific]. The hard constraints were [residency, RBAC at retrieval layer, sub-second p95]. We chose hybrid retrieval — graph + vector — because graph alone missed [X], vector alone missed [Y]. The architecture had three layers: orchestration, graph retrieval, vector retrieval. The biggest design trade-off was synchronous-with-latency vs. async-with-eventual-consistency; we chose synchronous because the customer's downstream workflow couldn't tolerate stale answers. The thing that almost broke us in eval was [specific failure mode] — we caught it because we'd built RAGAs faithfulness scoring as a *gate*, not a metric. What I'd change now: I'd push harder for the customer to invest in the eval set up front; we underbuilt it because of timeline pressure. From a safety lens, the RBAC-at-retrieval was the under-discussed but most-important architectural choice — the model can't leak what it can't see. From the executive lens: the value was reducing the time-to-confident-answer for the customer's operations team by [X]%, which was directly tied to a measurable downstream KPI."

Why: structured story, defensible choices, named trade-off with reasoning, real failure mode, honest "I'd change this now," safety lens explicit, executive lens explicit. **This is the answer the project deep-dive round wants for 40 minutes.**

---

## The patterns across grades

Across both worked examples, the moves that separate Strong Hire from Lean No:

1. **Specificity over generality.** Lean No answers say "we used vector databases." Strong Hire answers say "we chose FAISS over Pinecone because [specific reason]."
2. **Trade-offs named, not hidden.** Every architecture has trade-offs; pretending yours didn't is itself a Lean No signal. Strong Hire answers lead with the trade-off you accepted.
3. **Failure modes admitted.** Apic interviewers reward the candidate who says "this almost broke us" over the candidate who claims everything went perfectly. Imperfection is more credible than perfection.
4. **Safety lens surfaced unprompted.** A Strong Hire candidate names the safety/governance dimension of every architecture without being asked. Lean No candidates wait to be asked, and then bolt it on.
5. **Executive translation included.** A Strong Hire candidate closes the architecture story with the business value framing — what this is worth in measurable terms, who in the customer's org cares.
6. **Honest gaps.** Saying "I'd do this differently now" is not a weakness signal — it's a maturity signal. Saying "I wouldn't change anything" in an interview is a Lean No.

---

## Working memory

When you start a drill, ask yourself in 5 seconds:

- Is this answer *specific*? (If "yes" — proceed. If "I'm using buzzwords" — stop and restructure.)
- Have I named a *trade-off*? (If no — stop and add one.)
- Have I admitted a *failure mode* or "I'd do differently"? (If no — add one.)
- Did I surface the *safety lens* unprompted? (If no — add it.)
- Did I close with the *business value*? (If no — add it.)

Five questions. Internalize them. They are the difference between Hire and Strong Hire.
