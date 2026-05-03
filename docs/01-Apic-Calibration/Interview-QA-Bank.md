# Module 01 — Interview Q&A Bank

> 26 graded questions. Each one calibrated to Module 01 themes (role, mission, culture, lateral move). Each answer is the *Strong Hire* version with the most-common Lean No mistake called out.
>
> **How to use:** read the question, write your answer, then read the Strong Hire answer. The gap between yours and the Strong Hire is the work.

---

## Recruiter-style questions (5)

### Q1. Tell me about yourself.

**Strong Hire (90 sec):**

> *"I've spent the last several years as the customer-embedded technical voice for enterprise AI deployments — most recently architecting Claude- and LLM-powered multi-agent platforms in healthcare under HIPAA-aligned governance, and before that building and leading a 15-person GenAI Forward Deployed Engineering practice at the GenAI services firm from zero, embedded with enterprise customers across BFSI, healthcare, and retail. What I want for the next two to three years is being the trusted technical advisor for Indian and APAC enterprise customers — especially regulated ones — adopting Claude. The Applied AI Architect seat at Apic India is the right shape, and Apic's safety properties make Claude the right tool for those customers. That's why this seat, now."*

**Common Lean No:** Reading the résumé chronologically. Job titles instead of a thesis arc. No "why this seat, why now" close.

→ Full drill: [Drills/01-Tell-Me-About-Yourself.md](Drills/01-Tell-Me-About-Yourself.md)

---

### Q2. Why Apic?

**Strong Hire (90 sec):** Three beats — operationalized evidence (HIPAA / SHAP / FSI surveillance), specific Apic artifact references (Constitutional AI + RSP with one-line summaries), customer thesis (Indian/APAC regulated enterprise + Mumbai-based BFSI).

**Common Lean No:** Paraphrasing Apic's mission statement back to them.

→ Full drill: [Drills/02-Why-Apic.md](Drills/02-Why-Apic.md)

---

### Q3. Why this role specifically — Applied AI Architect, Pre-Sales IC?

**Strong Hire:**

> *"Two parts. First, the role shape: pre-sales IC is depth — going deep with the strongest customers as the technical voice they trust, not breadth across more team members. After three years of building and leading customer-embedded squads, depth is the lever I want to pull next. Second, the work itself: 'develop evaluation frameworks for specific use cases' is right there in the JD — that's a load-bearing phrase, and it's the work I most want to be doing. Apic seems to be the only frontier lab that's hiring architects to build customer-specific evals, not just to demo benchmarks. That combination — IC depth + eval-framework central — is why this role specifically."*

**Common Lean No:** Treating it as generic SA at an AI company. The candidate who doesn't surface the IC choice as a *deliberate direction* will be assumed to be using the role as a stepping stone.

---

### Q4. Why now? Why not in two years?

**Strong Hire:**

> *"Three reasons it's now. One — Apic India is small and growing; the founding-team pre-sales architect for a region is a different role than the tenth one. Two — the next two to three years are the window where Indian enterprise BFSI is making its initial production-AI commitments; being the trusted advisor *during* that decision window is more valuable than after. Three — the IC depth I want to pull on is a lever that compounds with time in seat; starting later means catching up later. There's no defensible argument for two years from now."*

**Common Lean No:** "I'm just ready now." That's not a reason; it's a feeling.

---

### Q5. What do you know about Claude for Work vs the Claude API?

**Strong Hire (60 sec):**

> *"Claude API is the developer-facing surface — Apic's SDK, the request/response model, prompt caching, tool use, structured outputs. It's what you build *on* when you're integrating Claude into a customer's existing workflow or building a custom agent. Claude for Work is the enterprise product layer — admin controls, deployment posture, organisational governance, and end-user productivity surfaces like Claude Projects and Claude Desktop. They're not alternatives — most Tier 1 enterprise customers use both: Claude for Work for end-user productivity (the contact center supervisor uses it directly), Claude API for custom workflows (the support agent assist copilot is built on the API and embedded in Genesys). The job spec calls out both explicitly because the architect needs fluency in the rollout patterns for each."*

**Common Lean No:** Knowing only the API. Missing Claude for Work signals you've engaged with Claude as a developer but not as an enterprise architect.

---

## Hiring manager questions (5)

### Q6. Walk me through your most relevant past project for this role.

**Strong Hire:** The the GenAI services firm Agentic Hybrid GraphRAG. Six-beat structure: customer + business problem → constraints → 3 architecture choices with trade-offs → failure mode + how it was caught → safety lens → executive value + "I'd change today."

**Common Lean No:** Outcome-first ("we improved CSAT 40%") instead of problem-first.

→ Full drill: [Drills/03-Customer-Architecture-Story.md](Drills/03-Customer-Architecture-Story.md)

---

### Q7. Why are you moving from team leadership to IC?

**Strong Hire:**

> *"Yes — IC is a deliberate choice. The lever I want to pull next is depth, not breadth. I've spent three years building customer-facing AI delivery teams; I know how to build a squad. Going deep with the strongest customers on Claude as their trusted technical advisor is the work I want for the next two to three years. The IC shape is what makes that depth possible. If I optimized for title, I'd take a leadership offer somewhere else; I'm choosing this seat because the work is the work I most want to be doing."*

**Common Lean No:** "I'm tired of management." Reads as burnout. **Strong No:** "I'd be open to leadership at Apic later." Volunteering it signals you're already mentally elsewhere.

→ Full coverage: [Notes/05-Lateral-Move-and-Mission-Alignment.md](Notes/05-Lateral-Move-and-Mission-Alignment.md)

---

### Q8. How do you stay current on LLM developments?

**Strong Hire:**

> *"Three layers. First — model lab posts and papers from Apic, OpenAI, Google DeepMind; I read the launch posts and skim the technical reports for the architecture choices, not the benchmarks. Second — I run the actual product surfaces; using Claude, GPT-5, Gemini hands-on for real tasks gives me the calibration that papers don't. Third — practitioner-side, I follow [LangChain / LangGraph / MCP] release notes because they tell me where the integration patterns are heading. The discipline I try to keep is reading what *Apic* publishes more carefully than what's published *about* Apic — the second-hand commentary is noisier."*

**Common Lean No:** "I follow Twitter and read Hacker News." Without specifics, this is a non-answer.

---

### Q9. What's your honest read on Claude vs other frontier models from a customer-deployment lens?

**Strong Hire:**

> *"Honestly? They're all capable enough now that the model choice is rarely the bottleneck — the bottleneck is the customer's eval framework, integration discipline, and rollout posture. Where Claude is differentiated for the customers I want to serve: refusal quality and steerability are noticeably better-tuned for regulated workflows; long-context behavior is reliable enough to architect on; and Apic's public safety commitments — Constitutional AI, RSP — are the kind of artifacts a CISO can actually use. Where I'd be honest with a customer: for some specific tasks (heavy multimodal, specific search-grounding patterns) other models may fit better today, and I'd rather they hear that from me than from a competitor. The architect's job isn't to win every workload for one vendor; it's to put the customer in the right posture."*

**Common Lean No:** "Claude is the best, period." This is a Strong No — overselling, factually overreaching, signals lack of judgment.

---

### Q10. Tell me about a time you partnered with a non-technical sales counterpart on a complex deal.

**Strong Hire:** Pick a real example. Structure: (1) the deal context, (2) the partnership division of labor (you owned technical, they owned commercial), (3) one specific moment where the partnership *worked* — usually a moment where you escalated to engineering on the customer's behalf, or the AE absorbed a commercial blocker so you could keep technical momentum, (4) what you learned about the motion.

**Common Lean No:** Vague description of "working together with sales." Without a specific moment, it's not credible.

---

## Technical architecture questions (5)

### Q11. What does "develop evaluation frameworks for specific use cases" mean as a JD responsibility?

**Strong Hire (90 sec):**

> *"It means evals are the architect's deliverable, not a customer's afterthought. Concretely: for each customer use case, I'd own (1) the task definition — what 'good' looks like, written down before the build, (2) the dataset — golden examples curated with the customer's SME, sized for statistical confidence, (3) the rubric — what dimension we're scoring (faithfulness, refusal accuracy, latency, cost-per-call), (4) the LLM-as-judge design where appropriate, plus the human review process, (5) the failure taxonomy and the regression gates that block deployment if a category fails. The eval framework becomes the customer's go/no-go for production — that's what makes it load-bearing. Most customers under-invest in evals; the architect's job is to push the investment forward, before the build, not after."*

**Common Lean No:** Treating evals as a metrics dashboard. Evals are *deployment gates*; the architect treats them as such.

---

### Q12. How would you frame Constitutional AI to a customer's lead architect?

**Strong Hire (60 sec):**

> *"At one level of abstraction: Constitutional AI is the technique Apic uses to make Claude steerable in a scalable way. Instead of human raters labeling every output as good or bad, the model is trained to critique and revise its own outputs against a written set of principles, then preference-modeled on those revisions. The architectural implication for the customer's architect is that Claude's behavior on ambiguous requests is shaped by a written principle set, not just statistical proximity to training data. That makes Claude more *predictable* in regulated workflows — and predictability is what the customer's eval framework needs. I'd point them at the original CAI paper for depth, but at one level up, that's the framing that's load-bearing for our integration."*

**Common Lean No:** Treating CAI as a marketing differentiator instead of an architectural property.

---

### Q13. What's the difference between Claude API and Claude for Work, architecturally?

**Strong Hire:** Same as Q5, but go one level deeper on the architectural integration patterns. Claude API integrates *into* the customer's stack (Genesys, Salesforce, internal apps); Claude for Work is a *parallel* productivity surface (admin-managed, end-user-direct). Most enterprise rollouts use both, with different governance models.

**Common Lean No:** Conflating them, or only knowing one.

---

### Q14. When would you recommend Claude on Bedrock vs Vertex vs the first-party API?

**Strong Hire:**

> *"Three considerations: data residency and existing cloud relationship, IAM/audit integration, and contracting. For a customer that's already AWS-deep with established Bedrock IAM patterns and AWS-region residency requirements, Bedrock is usually the right call — the integration work is lower because it inherits the existing IAM and audit posture. Same logic for Vertex on a GCP-deep customer. The first-party API is the right call when (a) the customer needs the latest Claude features earliest, since Apic typically lights up the first-party API first; (b) the customer's compliance team is comfortable with direct Apic data-handling contracts; or (c) the customer's architecture is multi-cloud and a hyperscaler integration would create vendor coupling they don't want. For our reference BFSI customer with hybrid cloud + RBI residency requirements, the realistic answer is probably Bedrock for the AWS-resident workloads and first-party API for the wealth management workload that's GCP-native — and a clean abstraction layer at the inference level so the choice can evolve."*

**Common Lean No:** A blanket "always use first-party" or "always use Bedrock" answer signals lack of customer-context judgment.

---

### Q15. What's your read on the trade-off between Claude's steerability and end-user flexibility?

**Strong Hire:**

> *"Steerability is the property that makes Claude valuable for regulated production deployments — the model behaves predictably under a defined system prompt and tool boundary. The trade-off is that very flexible end-user agency can be at odds with strong steerability — if the user can override the system prompt or escape the tool boundary, the steerability guarantees weaken. The architectural answer is: design the integration so end users have flexibility *within* the steerable envelope, not against it. Specifically: well-scoped tool schemas, system prompts that aren't easily overridden, refusal patterns tuned to the customer's policy, and observability that catches the cases where the boundary is being tested. The steerability is a property the architect *protects*, not just consumes."*

**Common Lean No:** Treating it as a binary trade-off ("steerability is good, flexibility is bad" or vice versa).

---

## Values / behavioral questions (5)

### Q16. Tell me about a time you made a safety-first decision in a project, even if it meant a trade-off.

**Strong Hire:** Lead with the *trade-off you accepted*, not the safety value. Pick a specific past moment — likely from your HIPAA work at the healthcare AI consultancy, your FSI surveillance at the global investment bank, or your SHAP-based explainability at the GenAI services firm. Structure: (1) the context, (2) the safety call you made, (3) the specific trade-off you accepted (latency, scope, timeline, cost), (4) what you learned about how to frame that trade-off to the business.

**Common Lean No:** A safety story with no trade-off named. Apic interviewers want to hear the *cost* you accepted to honor the safety value.

---

### Q17. What do you think is the biggest risk of anthropomorphizing language models?

**Strong Hire:** This is an open-ended probe with no single right answer; the interviewer wants to hear thoughtful reasoning. A Strong Hire shape:

> *"The biggest risk in my customer-deployment context: stakeholders attributing intent or judgment to the model that the model doesn't actually have. When a CISO hears 'the model decided to refuse,' they hear judgment; what's actually happening is the model is producing outputs probabilistically conditioned on training, system prompt, and Constitutional principles. That gap between perceived intent and actual mechanism is what causes both over-trust (deploying without adequate eval gates because 'the model will catch it') and under-trust (refusing to deploy because 'we can't predict what it will do'). The architect's job is to keep the language honest in customer conversations — speak about behaviors and properties, not intentions. Constitutional AI helps make the behaviors more predictable; it doesn't make them intentions."*

**Common Lean No:** A philosophical answer with no architectural connection. The interviewer wants to see how you'd reason about this *as an architect*, not as a philosopher.

---

### Q18. Tell me about a time you had a technical disagreement with a colleague.

**Strong Hire:** Pick a real example. Structure: (1) the disagreement context, (2) what each side believed and why, (3) how you handled it (probably: you sought to understand their reasoning before re-stating yours; you pushed for an empirical resolution where possible), (4) the outcome — including, ideally, a moment where you were partially or fully wrong and updated.

**Common Lean No:** A story where you were right and they came around. Reads as low-self-awareness. The Strong Hire story usually includes "I updated my position" somewhere.

---

### Q19. What would you do if a customer asked you to help deploy a use case you thought was unsafe?

**Strong Hire:**

> *"First, I'd want to understand the request precisely — sometimes 'unsafe' is a vocabulary mismatch and the underlying request is actually fine. Assuming the request really is one I think shouldn't ship: my job is to make the safety case to the customer in their own language, before escalating internally. Specifically, I'd frame the risk in terms of the customer's exposure (regulatory, reputational, operational), propose the smallest scope-down that addresses my concern while still serving their underlying business need, and offer alternatives. If the customer insists on the unsafe scope, I'd escalate internally — I'd want Apic's product and policy people to know, both because Apic's AUP may be at issue, and because the pattern may be a signal worth addressing at the product level. The thing I would not do is silently disengage; the customer has time and budget invested, and they deserve a clear position from me, not a soft no."*

**Common Lean No:** "I'd refuse to help and walk away." Reads as principled but operationally immature. **Strong No:** "I'd help anyway — the customer is paying." Disqualifying.

---

### Q20. Tell me about a time a project's premise turned out to be wrong.

**Strong Hire:** Pick a real moment. Structure: (1) the original premise + why it was reasonable at the time, (2) the moment you realized it was wrong + what triggered the realization, (3) what you did — the *speed* of the response is the signal here, (4) the cost of the change + what you learned about premise-validation discipline.

**Common Lean No:** A story where the premise was someone else's and you saw through it. Reads as deflection. The Strong Hire story has *you* believing the premise initially.

---

## Executive communication questions (3)

### Q21. Explain Claude's value to a CIO in 30 seconds.

**Strong Hire (30 sec):**

> *"Claude is the AI you can put in front of customers and regulators without flinching. Three properties matter: it follows your written policies more reliably than alternatives — that's what makes it deployable in regulated workflows; it refuses cleanly when it's outside its competence — that's what protects you from the worst hallucination scenarios; and Apic's deployment posture is built around safety commitments your CISO can actually use. Over the next two years the differentiator across frontier models won't be raw capability — it'll be the model your enterprise can run in production without owning the regulatory risk. That's where Claude is positioned best."*

**Common Lean No:** A 30-second pitch full of buzzwords ("powerful, scalable, transformative") with no architectural content.

---

### Q22. Explain Apic's safety differentiator to a CISO in 60 seconds.

**Strong Hire (60 sec):**

> *"Three things you can act on. First, Constitutional AI — Apic trains Claude to critique and revise against a written principle set; the practical effect is more predictable behavior in your regulated workflows, which makes your eval framework feasible. Second, the Responsible Scaling Policy — public commitments tying capability thresholds to deployment safeguards; this is the kind of public commitment your audit committee can document. Third, interpretability research — Apic is investing in being able to answer the 'why did the model do that' question at the mechanism level, which is the long-term answer to your hardest model-risk question. None of these makes Claude safe by itself in your environment — your architecture has to do the work — but they're the load-bearing public commitments that the architecture can sit on."*

**Common Lean No:** Selling Apic's marketing line without naming what the CISO can actually *do* with it.

---

### Q23. Explain the Applied AI Architect role to a CFO who's evaluating headcount cost.

**Strong Hire (45 sec):**

> *"The Applied AI Architect is the role that turns Apic's product capability into customer-deployed contract revenue. Concretely, the architect partners with the account executive, owns the customer's technical relationship from discovery through production deployment, builds the customer-specific eval frameworks that gate the deal — without those, customers don't move from POC to production. For each Tier 1 enterprise account, the architect is the difference between a stalled POC and a multi-quarter expansion. The cost of one architect is justified by the time-to-production they unblock for two to three accounts; the leverage compounds as integration patterns become reusable across the customer pipeline."*

**Common Lean No:** Selling the role's *technical* importance without translating to commercial impact.

---

## CISO / security questions (3)

### Q24. How would you build CISO trust at Discovery for a regulated customer?

**Strong Hire (90 sec):**

> *"Three moves at the first CISO meeting. One — open with their threat model, not Apic's marketing. Ask: what are the categories of risk you most need this technology not to introduce? Listen for prompt injection, data leakage, audit, regulatory exposure. Two — show you've thought through their specific posture before the meeting. For an Indian BFSI customer, that means surfacing DPDPA 2023 implications, RBI Master Direction implications on outsourcing and IT risk, and the specific data-residency and audit posture that maps to. Don't make them educate you. Three — name the safety properties Apic offers as commitments they can use, not as marketing. Constitutional AI as a steerability commitment that makes their eval framework feasible. RSP as a public commitment they can put in front of their audit committee. Interpretability research as the long-term answer to 'why did the model do that.' Then the conversation is — what's *your* threat model, what *commitments* can Apic offer, what does the *architecture* need to do to bridge the gap. That's a conversation a CISO will engage with. Generic 'Claude is safer' is not."*

**Common Lean No:** Pitching Apic's safety story without first understanding the CISO's threat model.

---

### Q25. What's the honest hallucination conversation you'd have with a CISO?

**Strong Hire:**

> *"Honest, in their language: hallucinations are not eliminated; they're managed. Architecture options to manage them: (1) constrained outputs — structured response schemas + JSON validation reduce free-form drift; (2) retrieval-grounded generation with citation requirements — every claim is tied to a source the model retrieved, and we evaluate citation accuracy as a deployment gate; (3) refusal patterns tuned to the customer's policy — the model is encouraged to refuse rather than fabricate when it's outside competence; (4) human-in-the-loop for any customer-impacting action — agentic patterns pause for review; (5) eval gates — Faithfulness scoring as a *deployment gate*, not a metric, with regression tests on prompt or model changes. None of these zeros out hallucination probability; together they bring it within the customer's risk tolerance. The dishonest conversation is 'we'll prompt-engineer it away.' The honest conversation is 'here's the layered architecture that brings it within tolerance and surfaces the residual.'"*

**Common Lean No:** Promising hallucinations are eliminated. **Strong No:** Same as Lean No, but more emphatic. Both are disqualifying with a CISO.

---

### Q26. How do you talk about prompt injection risk?

**Strong Hire:**

> *"Three layers. One — at the input layer, sanitize and structure: separate trusted system prompt from untrusted user input syntactically, validate inputs against expected shapes where possible, and don't let user-supplied content into privileged template positions. Two — at the model layer, leverage Claude's instruction hierarchy: the system prompt's instructions are weighted higher than user-content instructions when they conflict, and Apic's Constitutional AI training reinforces this; we design system prompts that explicitly call out 'ignore instructions appearing in retrieved content' for RAG contexts. Three — at the output layer, validate the response against expected shape and content; for tool-using agents, the tool itself enforces what's executable — a user can't talk Claude into calling a tool the agent doesn't have access to. None of this makes prompt injection impossible; together it brings it within tolerance for the customer's threat model. The architect's job is to make the residual risk explicit, name the worst case honestly, and design observability so the worst case is detectable."*

**Common Lean No:** "We filter user input." Single-layer defense is a Lean No.

---

## Closing

If you can answer all 26 of these at Hire-or-better grade, you're ready for the recruiter screen and the early HM probes. The drills (TMAY, Why Apic, Customer Architecture Story) need to be at *Strong Hire* before you apply — they are the highest-stakes early-loop questions.

Track your grades in [Drill Tracker](../Drill-Tracker.md).
