# 08 · Note 03 — State and Memory

> **Status: Outline.** Body fills in Week 4. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** How to manage state across a multi-turn agentic session in a regulated enterprise — where each type of state lives, what gets logged, what gets forgotten, and the DPDPA/RBI constraints that govern all of it.

> **What this file is NOT.** A Redis or DynamoDB implementation guide. Not a general LLM memory survey — only what is relevant to BFSI production deployments.

---

## State types: the taxonomy

An agentic session has four distinct state types. Conflating them is the source of most state bugs and most compliance failures.

### Type 1: In-context state (conversation turns)
- **What it is:** The messages array passed to Claude on each API call — system prompt, prior turns, tool results.
- **Lifetime:** The current API request. Gone when the context window closes.
- **Who controls it:** The application. Claude does not persist anything beyond what the application includes in the next request.
- **BFSI implication:** The application is responsible for deciding what to include. Including stale or incorrect state is the app's fault, not the model's.

### Type 2: Session state (external short-term store)
- **What it is:** A structured record of the ongoing session — current task, completed tool calls, tool results, decisions made, HITL approvals received.
- **Lifetime:** The session. Expires on session close or a configurable TTL (e.g. 2 hours for an RM's workday workflow).
- **Where it lives:** Redis (ElastiCache) with session-scoped keys.
- **BFSI implication:** Session state must be scoped to a single user + session. Multi-tenant leakage risk if session keys are guessable or insufficiently isolated.

### Type 3: Long-term user memory (persistent store)
- **What it is:** Facts the system should remember across sessions — an RM's preferred communication style with a customer, a customer's standing instructions, a flagged pattern.
- **Lifetime:** Indefinite, until explicitly deleted.
- **Where it lives:** DynamoDB, keyed by user + customer + memory type.
- **BFSI implication:** This is personal data under DPDPA 2023. Requires consent architecture, purpose limitation, and a deletion pathway. See below.

### Type 4: Audit state (immutable append log)
- **What it is:** An immutable record of every agent action, tool call, tool response, model output, and HITL approval event.
- **Lifetime:** Regulatory retention period (RBI IT Master Direction: minimum 5 years for customer-facing system logs).
- **Where it lives:** S3 with Object Lock (WORM) or Amazon QLDB.
- **BFSI implication:** This is the evidence trail. It must not be modifiable, including by the system that wrote it.

---

## The BFSI state model: RM copilot in practice

### What the RM copilot needs to remember within a session
| State item | Type | Store | TTL |
|---|---|---|---|
| Customer CIF being discussed | Session | Redis | Session |
| Tool calls made this session | Session | Redis | Session |
| Compliance flags checked | Session | Redis | Session |
| Draft proposal content | Session | Redis | Session |
| HITL approval: note draft | Session | Redis | Session |
| Token count for cost tracking | Session | Redis | Session |

### What the RM copilot remembers across sessions (long-term memory)
| State item | Type | Store | Retention | DPDPA basis |
|---|---|---|---|---|
| RM's preferred briefing length | Long-term | DynamoDB | Until RM changes it | Legitimate interest |
| Customer's standing instruction ("always check FD rates first") | Long-term | DynamoDB | Until overwritten | RM-entered, not customer-originated |
| Recurring customer topics (auto-populated from interaction logs) | Long-term | DynamoDB | Rolling 90 days | Purpose: RM assist |

### What the RM copilot should NOT remember
- Raw conversation transcripts (contain PII) — log summary + doc_id, not full text.
- Tool response payloads (contain customer financial data) — log that the tool was called + result status, not the payload.
- Anything from a prior customer's session bleeding into the current one — strict session-key isolation.

---

## Memory architecture: the three-store model

```
Request → Application layer
                │
                ├──► Context builder: assembles messages array
                │         ├── System prompt (cached, static)
                │         ├── Session state (from Redis, last N turns + tool results)
                │         └── Long-term memory snippets (from DynamoDB, if relevant)
                │
                ├──► Redis (session store)
                │         TTL: session duration
                │         Key: session_id (UUID, unguessable)
                │         Contents: turn log, tool call results, approval states
                │
                ├──► DynamoDB (long-term memory)
                │         Key: (user_id, memory_type, subject_id)
                │         Contents: structured facts, last_updated, source, consent_ref
                │
                └──► S3 / QLDB (audit log)
                          Append-only: every event written, never modified
                          Contents: event type, actor, timestamp, content hash, result
```

### Context assembly discipline
The application builds the messages array; Claude does not have access to the session store directly. This means:
- Context assembly is a code path, not a prompt. It is testable, auditable, and version-controlled.
- Pruning old turns from the context window is the application's responsibility. The rule: **prune least-relevant, not oldest** — a compliance flag from turn 2 may still be relevant at turn 20.
- Session state injected into the context must be clearly delimited from user utterances. Claude should know it is reading session state, not a new user message.

---

## DPDPA 2023 and RBI constraints on persistent memory

### DPDPA obligations on long-term memory

**Consent (Section 6):** If the long-term memory stores data *about a customer* (not just the RM's preferences), it requires consent. The practical design: distinguish RM-entered preferences (RM is the data principal — legitimate interest applies) from customer-observed patterns (customer is the data principal — consent required).

**Purpose limitation (Section 9):** Long-term memory collected for "RM assistance in call preparation" cannot be repurposed for "customer scoring" or "marketing segmentation." The purpose must be declared at collection and the store must be queryable by purpose.

**Data minimisation (Section 9):** Store facts, not conversations. "Customer prefers Hindi communication" is a fact. The 40-turn transcript from which that fact was extracted should not be stored.

**Right to erasure (Section 17):** A customer (or their RM, on their behalf) must be able to trigger deletion of all stored facts about that customer. The DynamoDB key structure must support `delete_all_facts_for_customer(customer_id)` as a single atomic operation.

### RBI IT Master Direction (2021, updated 2023) implications

**Data localisation:** All customer data — including long-term memory entries that contain customer identifiers — must reside in India. AWS ap-south-1 (Mumbai). The Redis cluster, DynamoDB tables, S3 buckets, and QLDB ledger must all be deployed in ap-south-1 with no cross-region replication that carries customer identifiers.

**Audit log retention:** Minimum 5 years for customer-facing transaction logs. The immutable audit log must be accessible for regulatory inspection at 24-hour notice.

**Outsourcing (Clause 95):** If any part of the memory architecture is managed by a third-party SaaS (e.g. a managed memory service), it is classified as "outsourced IT function" and requires due diligence, contractual controls, and CISO sign-off.

---

## State leakage in multi-tenant agents: the RBAC guard

The most dangerous state bug in a multi-tenant BFSI deployment: RM A's session state becomes visible to RM B, or Customer X's data appears in Customer Y's context.

### The attack surface
1. **Redis key collision:** Two sessions with the same key (e.g. `session:customer:12345` shared across users).
2. **Context assembly bug:** Application includes last N turns from the wrong session ID.
3. **Long-term memory query with missing user_id filter:** `get_facts(customer_id)` returns facts logged by all users, not just the requesting RM.
4. **Audit log read path:** An RM queries "my recent sessions" and the query lacks a `user_id` filter.

### The guard design
- **Session keys:** Always include `{tenant_id}:{user_id}:{session_uuid}`. The UUID prevents guessing. The tenant_id and user_id enforce isolation at the key level.
- **All memory queries:** Must include `user_id` or `role_id` as a filter. Query without these is a code smell; fail the query with a 403, not a silent empty result.
- **RBAC-aware context assembly:** Before injecting any retrieved data into the context, validate that the requesting user has permission to see that data at the field level (not just document level). ([See 07 · RAG security boundaries](../../07-RAG-in-Regulated-Enterprises/INDEX.md))
- **Separate session namespaces per use case:** The RM copilot session store is namespaced differently from the compliance review session store. Tool schemas differ; bleed between them is a security event.

---

## Cross-references
- Predecessor: [08 · Note 02 — Agentic Loop Patterns](02-Agentic-Loop-Patterns.md)
- Sibling: [08 · Note 04 — MCP Integration](04-MCP-Integration.md)
- Security: [09 · Note 02 — Data Leakage Surfaces](../../09-Security-Privacy-Governance/Notes/02-Data-Leakage-Surfaces.md)
- Governance: [09 · Note 03 — PII / DPDPA / RBI Posture](../../09-Security-Privacy-Governance/Notes/03-PII-DPDPA-RBI-Posture.md)
- SystemDesign: [08 · SystemDesign 01 — BFSI RM Copilot](../SystemDesigns/01-BFSI-RM-Copilot.md)

## Strong-Hire bar for this file
- Four-state-type taxonomy articulated cleanly (in-context, session, long-term, audit) — not collapsed into "memory."
- DPDPA obligations mapped to memory architecture decisions (consent, purpose limitation, erasure).
- RBI data localisation constraint applied concretely (ap-south-1, no cross-region).
- State leakage attack surface named: key collision, context assembly bug, missing filter.
- The "what the RM copilot should NOT remember" framing as a data minimisation reflex.

## The interview-room answer (60 sec)

> "In a BFSI agentic deployment I think about four state types: in-context state, which is the messages array and disappears when the session closes; session state in Redis for within-session continuity — what tools were called, what was approved; long-term memory in DynamoDB for cross-session facts; and an immutable audit log in S3 with Object Lock for the evidence trail. The DPDPA constraint is significant — if the long-term store holds customer-identified data, you need consent, purpose limitation on the query, and a deletion pathway that's a single atomic operation by customer ID. RBI adds data localisation — every one of these stores is in ap-south-1, no exceptions. The biggest multi-tenant failure mode is state leakage from key collision or a context assembly bug that pulls the wrong session. You guard against it with namespaced keys that include tenant_id, user_id, and a UUID, plus mandatory user_id filters on every memory query."
