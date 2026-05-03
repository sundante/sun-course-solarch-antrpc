# 08 · Note 05 — Human-in-the-Loop & Decision Rights

> **Status: Outline.** Body fills in Week 4. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this file is.** The architecture of HITL approval flows — not a UX pattern, but a system design decision that determines audit trail, liability, rollback, and what happens when the human is wrong.

> **What this file is NOT.** A UX design guide for approval interfaces. Not a general discussion of "keeping humans in the loop" as a principle — this is the mechanical design that makes that principle real in production.

---

## Decision rights taxonomy

Not every Claude action requires HITL. Getting this wrong in either direction causes problems:
- **Too much HITL:** Every Claude output requires human approval. The RM is doing more work, not less. The system fails to deliver value.
- **Too little HITL:** Claude takes irreversible actions without approval. In BFSI, this is a regulatory event.

The right framework is a decision rights taxonomy — classifying every action the agent can take into one of three categories:

### Category A: Claude decides (no HITL)
- Claude takes the action directly, no human approval required.
- Valid for: **read-only, non-PII, reversible, low-stakes actions.**
- BFSI examples: fetching customer 360, retrieving SOP passages, running a named Snowflake query, looking up product eligibility.
- Criterion: if Claude returns a wrong result, the consequence is a bad recommendation to the RM — who is still in the loop before any customer-facing action.

### Category B: Claude drafts, human approves
- Claude produces a proposed action or output. A human reviews and approves before execution.
- Valid for: **write actions, customer-visible outputs, actions that generate audit events.**
- BFSI examples: drafting a CRM interaction note (RM approves before logging), drafting a proposal (RM approves before sending), creating a service request (RM confirms request details).
- Criterion: the action creates a permanent record or is visible to the customer.

### Category C: Human decides, Claude assists
- The decision is outside Claude's authority regardless of quality. Claude provides analysis and options; a credentialed human decides.
- Valid for: **regulatory submissions, credit decisions, exceptions to policy, actions with legal consequences.**
- BFSI examples: compliance review submission to regulator (compliance officer signs off, not just approves Claude's draft), credit limit exceptions, AML escalations.
- Criterion: the decision carries personal legal liability for the person making it. Claude cannot hold liability.

---

## The BFSI decision rights matrix

| Action | Category | HITL type | Approver |
|---|---|---|---|
| Fetch customer 360 | A — Claude decides | None | — |
| Search SOP knowledge base | A — Claude decides | None | — |
| Get portfolio snapshot | A — Claude decides | None | — |
| Check compliance flags | A — Claude decides | None | — |
| Run named Snowflake query | A — Claude decides | None | — |
| Draft interaction note | B — Claude drafts | RM review + approve | RM |
| Draft product proposal | B — Claude drafts | RM review + approve | RM |
| Create service request | B — Claude drafts | RM confirm details | RM |
| Flag a compliance gap in document | B — Claude drafts | Compliance officer review | Compliance officer |
| Submit compliance review to regulator | C — Human decides | Human signs off | Compliance officer (named) |
| Approve credit exception | C — Human decides | Human decides | Credit committee |
| AML escalation | C — Human decides | Human decides | Compliance + Legal |

---

## HITL architecture: the mechanics

### The interrupt model
When Claude proposes a Category B action, the agentic loop pauses. This is not a UX pause — it is an architectural state transition.

```
Claude emits: tool_use { name: "log_crm_interaction", input: { ...draft... } }
Application: detects write tool → does NOT execute → transitions to PENDING_APPROVAL state
Application: surfaces draft to RM in UI
RM: reviews → approves | rejects | edits
If approved: application executes tool → logs approval event → resumes loop
If rejected: application sends rejection reason back as tool_result → loop continues
If edited: application applies RM's edits to the draft → executes the edited version → logs original + edited + approval
```

### Timeout behavior
What happens if the RM doesn't respond? This is a system design question, not a UX question.

Options:
- **Expire and require re-initiation:** After N minutes with no approval, the pending action expires. The session is still alive; the RM can request a new draft. This is the safer default.
- **Escalate:** After N minutes, notify the RM's supervisor that an approval is pending. Used for time-sensitive workflows (compliance review with a filing deadline).
- **Auto-reject:** If no approval in N minutes, treat as rejected. The action is logged as attempted but not executed.

The BFSI default: **expire and require re-initiation**. Do not auto-approve on timeout. In a regulated environment, a silent approval is a liability.

### The approval UI requirements
The RM must see, at minimum:
1. **What Claude is proposing to do** — the exact tool name and the complete payload, not a summary.
2. **Why Claude is proposing it** — the reasoning trace (the prior turn context that led to this draft).
3. **What the consequence is** — "This will be logged to the CRM and visible to all RMs and compliance teams."
4. **Edit capability** — the RM must be able to edit the draft before approving. Approving a forced binary removes accountability.
5. **A distinct reject pathway** — "Reject with reason" so the rejection is logged alongside the draft.

---

## Audit log requirements for HITL

Every HITL approval event must generate an immutable audit record containing:

| Field | Value |
|---|---|
| `event_type` | `hitl_approval` |
| `session_id` | UUID |
| `actor_id` | RM's SSO identity (not "the system") |
| `actor_role` | Role at time of approval |
| `timestamp` | ISO 8601, millisecond precision |
| `draft_hash` | SHA-256 of the original Claude draft |
| `approved_version_hash` | SHA-256 of the version actually executed (may differ if edited) |
| `edit_delta` | Structured diff between draft and approved version (if edited) |
| `action_executed` | The exact tool call that was executed post-approval |
| `model_version` | Model ID that produced the draft |
| `system_prompt_version` | System prompt commit hash |

**Why the hash matters:** The audit record does not store the full draft text (that's PII risk). It stores a hash. If the draft text needs to be produced for a regulatory inquiry, it can be reconstructed from the session state log + the hash confirms authenticity.

**Why the edit delta matters:** RMs may approve Claude's draft with edits. The compliance record must show what Claude proposed and what the RM changed. This is the difference between "Claude logged this note" and "RM approved this note with these modifications."

---

## The CISO question: what if Claude is wrong and the human approved it anyway?

This is the question every CISO asks. The honest answer has three parts:

**Part 1: The audit log shows exactly what happened.** The CISO can see: Claude's draft, the RM's approval (or edits), the time elapsed between draft and approval (a 0.3-second approval of a complex note is a flag), and the model version. There is no "Claude did it" without a human signature next to it.

**Part 2: The architectural response is post-approval validation, not pre-approval perfect models.** For high-stakes Category B actions (compliance gap flags, proposal drafts), the architecture includes a downstream validator that runs after approval: a deterministic rule-check against the proposed action, or a second-model review for compliance content. This is not asking Claude twice — it's asking a rule engine that cannot hallucinate.

**Part 3: Rollback design.** For reversible write actions (CRM notes), the system design includes a "void" capability — the RM can void an approved note within a time window (e.g. 30 minutes), which creates a `void_event` in the audit log rather than deleting the original record. Immutable logs, mutable business state.

The CISO-level framing: "HITL does not guarantee correctness. It guarantees accountability. The RM who approves an incorrect note owns the correction."

---

## Cross-references
- Predecessor: [08 · Note 04 — MCP Integration](04-MCP-Integration.md)
- Security: [09 · Note 04 — Audit Logging & Decision Rights](../../09-Security-Privacy-Governance/Notes/04-Audit-Logging-and-Decision-Rights.md)
- Security: [09 · Note 05 — CISO Trust-Building](../../09-Security-Privacy-Governance/Notes/05-CISO-Trust-Building-and-Honest-Hallucination.md)
- SystemDesign: [08 · SystemDesign 02 — BFSI Compliance Review](../SystemDesigns/02-BFSI-Compliance-Review-Agentic-with-HITL.md)
- Case Study: [08 · Case Study 01 — BFSI Agentic Workflow Walkthrough](../Case-Study/01-BFSI-Agentic-Workflow-Walkthrough.md)

## Strong-Hire bar for this file
- Three-category taxonomy (Claude decides / Claude drafts / Human decides) articulated as a *design tool*, not a checkbox.
- Timeout behavior described architecturally — "expire and require re-initiation" as the BFSI default with reasoning.
- Audit record fields named: draft hash, approved version hash, edit delta — not just "log the event."
- The CISO question answered in three parts: audit trail shows what happened, post-approval validation, rollback design.
- "HITL guarantees accountability, not correctness" as the honest framing.

## The interview-room answer (60 sec)

> "HITL is a system design decision, not a UX checkbox. I start with a decision rights matrix: Category A — Claude acts directly, for read-only non-PII calls. Category B — Claude drafts, human approves before execution, for any write action or customer-visible output. Category C — human decides with Claude assisting, for anything with personal legal liability. The mechanics: when Claude proposes a write tool call, the loop pauses — doesn't execute, transitions to pending approval. The RM sees the exact payload, the reasoning trace, and an edit option. If they don't respond, the action expires — we never auto-approve on timeout in a BFSI context. The audit record captures both the draft hash and the approved-version hash plus any edit delta, so the CISO can see exactly what Claude proposed and exactly what the RM signed off on. When the CISO asks 'what if Claude is wrong and the human approved it?' — the audit trail shows who approved and when, downstream validation catches structural errors, and reversible writes have a void window. HITL guarantees accountability, not correctness."
