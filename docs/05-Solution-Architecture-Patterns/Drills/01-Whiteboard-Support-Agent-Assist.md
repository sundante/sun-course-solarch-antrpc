# 05 · Drill 01 — Whiteboard Support Agent Assist

> **Status: Outline.** Body fills in Week 3. Voice: principal-level, BFSI-threaded, adversarial probing.

> **What this drill is.** A 20-minute whiteboard of the Customer Support Agent Assist architecture — the most common use case in BFSI pre-sales. The probe question: how does Claude fit in the existing Genesys + Salesforce stack without ripping anything out?

> **What this drill is NOT.** A "describe the ideal chatbot" exercise. The interviewer is probing whether you can defend specific architectural decisions under pressure from a skeptical senior engineer who has seen bad AI deployments fail.

---

## Prompt

> "You're in a pre-sales meeting at a large Indian BFSI institution. The VP of Customer Operations has asked you to sketch the architecture for a Claude-based agent assist system for their contact center. They already run Genesys for CTI and Salesforce for CRM. They have 2,000 agents handling 200,000 calls per day. The CISO is in the room. You have 20 minutes. Go."

---

## Time box

**20 minutes.** Use the first 2 minutes to ask 2–3 clarifying questions before drawing a single box. Use the last 2 minutes to call out the top 2 risks proactively — before the interviewer does.

---

## Clarifying questions to ask (Strong-Hire behavior)

Before picking up the marker, ask:

1. "What's the primary metric you're trying to improve — handle time, first-call resolution, agent satisfaction, or all three?"
2. "Are you looking for Claude to assist the agent in real-time during the call, or as a post-call summarization tool, or both?"
3. "What's the data residency requirement — is all customer data currently on-prem or in cloud, and if cloud, which region?"

Why these questions? They constrain the architecture before you draw it. Handle time reduction → real-time inference → streaming + Sonnet/Haiku. Post-call summarization → batch → Sonnet, async. Data residency → Bedrock ap-south-1, mandatory.

---

## HLD sketch (describe verbally as you draw)

### The boxes (left to right)

**Box 1: Genesys CTI**
- The call arrives at the contact center.
- Genesys handles the telephony, the queue, and the agent desktop.
- Genesys Event Streaming emits a real-time transcript stream (WebSocket or Server-Sent Events).

**Box 2: Transcript Ingestion Service**
- A thin AWS Lambda or container service subscribed to the Genesys transcript stream.
- Receives each customer utterance as it completes.
- Triggers the orchestration service.

**Box 3: Orchestration Service**
- The central coordinator. Owns the conversation state for the current call.
- Validates the agent's IAM/SSO token → extracts role, branch, region, department.
- Constructs the Claude prompt: cached policy bundle (system) + conversation history + current utterance.
- Calls the Claude API (via Bedrock ap-south-1).
- Parses the response.
- Calls Salesforce (read) for customer context if needed.
- Returns the suggestion to the Agent Desktop.

**Box 4: Claude (via Bedrock ap-south-1)**
- Model: Sonnet 4.6 (balanced quality + latency for interactive path).
- System prompt: 30K-token BFSI policy bundle, cached.
- Tool calls: read-only Salesforce lookup (account info, interaction history).
- Output: structured suggestion (next best action, relevant policy snippet, escalation flag).

**Box 5: Salesforce CRM**
- Read path: customer 360 data retrieved via Salesforce REST API on each call.
- Write path: post-call note, recommended next action — written after agent confirms (HITL gate).

**Box 6: Agent Desktop (Genesys embedded app)**
- Shows Claude's suggestion in a side panel.
- Agent sees: suggested response, relevant policy reference, escalation recommendation.
- Agent acts: uses, ignores, or edits the suggestion.
- Agent confirms: for write actions (Salesforce update), explicit click to confirm.

**Box 7: Audit & Observability**
- Every Claude call logged to the JSONL audit store (S3 or the enterprise's SIEM).
- RBAC context logged with every entry.
- Eval harness samples 5–10% of logs daily.

**Arrows to narrate:**
- Genesys → Transcript Service: real-time transcript stream.
- Transcript Service → Orchestration: trigger per utterance.
- Orchestration → Claude (Bedrock): prompt with cached system + dynamic context.
- Orchestration → Salesforce: read tool call (customer data).
- Claude → Orchestration: suggestion (structured JSON).
- Orchestration → Agent Desktop: display suggestion.
- Agent Desktop → Orchestration: confirmation event (for write actions).
- Orchestration → Salesforce: write tool call (post-confirmation).
- Orchestration → Audit Store: every interaction logged.

---

## 8 probe questions with strong-hire answers

### Probe 1: "Why Sonnet and not Haiku? Haiku is 5× cheaper."

**Weak answer:** "Sonnet is more accurate."

**Strong-Hire answer:** "Haiku is the right choice for the routing and classification step — deciding whether this utterance requires a Claude synthesis response at all. Sonnet handles the synthesis. At 200K calls/day with a 30K-token system prompt, the dominant cost is the cache write, not the inference model. The differential between Sonnet and Haiku on the ~150-token dynamic input is small compared to the cache economics. And Sonnet's improved reasoning on policy lookup is measurably better in the eval for this use case. If the eval shows Haiku parity, we switch."

---

### Probe 2: "The CISO says no PII in any external API call. How do you handle it?"

**Weak answer:** "We'll anonymize the data."

**Strong-Hire answer:** "The customer's name, account number, and transaction data never leave the Salesforce API in raw form. The orchestration service applies a token substitution: internal reference IDs replace PII identifiers before they enter the Claude prompt. Claude never sees the customer's name — it sees 'Customer #CID-8837'. The Salesforce write back uses the internal ID to update the right record. The mapping table (CID → real customer ID) lives in the orchestration service's memory, not in any log. DPDPA 2023 defines this as pseudonymization, which is an acceptable privacy control for data processor contexts."

---

### Probe 3: "What happens when Claude's latency spikes and the agent is waiting?"

**Weak answer:** "We set a timeout."

**Strong-Hire answer:** "Streaming is the first defense — the agent sees the first tokens within 300–500ms even if the full response takes 2 seconds. The second defense is a circuit breaker: if Claude P95 latency exceeds 3× the SLA for more than 60 seconds, the orchestration service stops calling Claude and displays a fallback message in the agent panel: 'AI suggestions temporarily unavailable — contact your supervisor for assistance.' The third defense is model tiering — if Sonnet is slow, we have a fallback config that routes to Haiku for the duration of the incident. The agent experience degrades gracefully; it doesn't freeze."

---

### Probe 4: "How does RBAC work? An agent in one branch shouldn't see another branch's customer data."

**Weak answer:** "We use Salesforce sharing rules."

**Strong-Hire answer:** "Two controls, both independently enforced. First, the Salesforce API query is parameterized with the agent's branch and region, derived from their SSO token. Salesforce sharing rules provide a second layer — even if the query parameter were wrong, the sharing rules prevent out-of-scope data from being returned. But we don't rely on the second layer — the first is our enforced control. The agent's role context is injected into the system prompt as a system-controlled field, not as user input: 'This agent is authorized for: Branch 42, Region: Maharashtra, Role: Tier 1 Agent.' Claude's instructions explicitly state this context cannot be overridden by the human turn."

---

### Probe 5: "What's your eval framework? How do you know the suggestions are correct?"

**Weak answer:** "We test it manually before launch."

**Strong-Hire answer:** "Four layers. Pre-launch: a shadow-mode deployment where every production request is sent to Claude and scored by an LLM-as-judge harness. We don't go to Phase 2 unless pass rate ≥ 90% on factual accuracy, policy adherence, and escalation correctness. Post-launch: 5% of production traffic is continuously sampled and scored daily. If the pass rate drops 2% from baseline, we get an alert. If it drops 5%, we trigger an incident. The eval criteria are use-case-specific — not generic 'helpfulness' metrics but BFSI-specific: does the policy reference match an actual RBI circular? Was the escalation trigger fired correctly? The judge model is Haiku — cheap, consistent, auditable."

---

### Probe 6: "What if a customer asks the agent about something that's in the news — a fraud event at the bank — that isn't in the policy bundle?"

**Weak answer:** "Claude will say it doesn't know."

**Strong-Hire answer:** "The system prompt explicitly scopes Claude's knowledge domain: 'You may only answer questions grounded in the provided policy bundle or in information retrieved via tools. If a question cannot be answered from these sources, respond with: This topic requires escalation to a supervisor.' The agent gets a clear, honest signal to escalate — not a hallucinated answer about a fraud event. This is better for both the customer and for the bank's risk posture. An AI that confidently makes up answers about a live fraud event is worse than an AI that escalates. We tune the scope narrowly and err toward escalation."

---

### Probe 7: "Genesys upgrades their API next quarter. What breaks in your design?"

**Weak answer:** "We'll update the integration."

**Strong-Hire answer:** "The Genesys integration is isolated in the Transcript Ingestion Service — a single module with a clear contract: it receives a transcript stream and emits a structured event to the orchestration service. When Genesys upgrades, only that module needs to change. The orchestration service, Claude integration, and Salesforce integration are unaffected. This is the thin orchestration layer principle: each integration point is a seam, not a dependency. We also maintain a Genesys SDK version pin and a change notification agreement with the customer's Genesys team — we need 30 days notice of breaking changes."

---

### Probe 8: "This is fine for Tier 1 support. What about Tier 2 — can Claude handle it?"

**Weak answer:** "We can upgrade to Opus for Tier 2."

**Strong-Hire answer:** "Tier 2 support in BFSI typically involves complex cases: disputes, fraud investigations, loan restructuring. Claude can assist but the HITL posture changes. For Tier 1, the agent can act on Claude's suggestion directly for low-risk queries. For Tier 2, every Claude-assisted recommendation must be reviewed by the agent and, for credit/compliance-adjacent actions, by a second approver. The model choice also changes — Sonnet for synthesis is appropriate for Tier 1; for Tier 2 cases that involve reasoning over regulatory documents, we evaluate Opus. We prove this in the eval harness before deciding — not by assumption."

---

## Common traps

| Trap | What it signals |
|---|---|
| Drawing the architecture before asking clarifying questions | You have a pre-canned diagram, not a customer-specific solution |
| Proposing Claude replaces Genesys or Salesforce | You haven't understood the integration boundary doctrine |
| Saying "we'll add RBAC later" | You don't understand BFSI regulatory requirements |
| Skipping the eval framework | You don't treat quality as an architecture concern |
| Not mentioning the CISO proactively | You haven't read the room |
| Promising <200ms end-to-end with tool calls | You haven't done the latency math |

---

## Rubric

**Strong Hire:** Asks 2–3 clarifying questions before drawing. Articulates integration posture (bolt-on) clearly. Names all HLD components and their contracts. Answers all 8 probes with specifics. Proactively raises the top 2 risks (data residency, HITL). Cost model is at least directional.

**Hire:** Most of the above but misses one probe (acceptable if acknowledged). May need prompting to discuss the eval framework.

**Lean No:** Draws first, asks no questions. Diagram is generic (boxes labeled "AI" and "Database"). Doesn't mention RBAC until prompted. Eval framework is "we'll test it."

**Strong No:** Proposes replacing Genesys or Salesforce. Claims <100ms end-to-end without streaming. Doesn't mention HITL. Dismisses CISO concerns.

---

## Cross-references
- Note: [01 — Reference Architecture Anatomy](../Notes/01-Reference-Architecture-Anatomy.md)
- Note: [02 — Integration Boundary Doctrine](../Notes/02-Integration-Boundary-Doctrine.md)
- Note: [03 — Failure Modes Catalog](../Notes/03-Failure-Modes-Catalog.md)
- Note: [04 — Cost & Latency Modeling Primer](../Notes/04-Cost-and-Latency-Modeling-Primer.md)
- System Design: [Customer Support Agent Assist](../SystemDesigns/01-Customer-Support-Agent-Assist.md)

## Strong-Hire bar for this drill
- Asks before drawing.
- All 8 probe answers are specific, not generic.
- RBAC answer distinguishes first-layer enforcement from second-layer backup.
- Eval framework answer names specific criteria, not generic quality metrics.
- Proactively calls out the CISO's concerns before being asked.
