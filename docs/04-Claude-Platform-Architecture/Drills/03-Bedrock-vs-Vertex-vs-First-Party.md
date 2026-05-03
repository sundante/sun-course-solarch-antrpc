# 04 · Drill 03 — Bedrock vs Vertex vs First-Party

> **Status: Outline.** Body fills in Week 2. Voice: principal-level, BFSI-threaded, Apic-calibrated.

> **What this drill is.** Pick the deployment surface for an Indian BFSI customer and defend the choice for 5 minutes against a skeptical CIO + CISO simulated panel.

> **What this drill is NOT.** A vendor recommendation. Not a cloud sales pitch.

---

## Prompt

> *"You're at a decision meeting with a large Indian BFSI customer. Their primary cloud is AWS Mumbai (~80% of workloads), but they have a strategic GCP relationship growing in their data team (~15%). Recently a new CIO has signaled willingness to evaluate first-party SaaS for AI workloads. The CISO wants to know which Claude deployment surface you'd recommend, and the CIO wants procurement story + roadmap velocity. You have 5 minutes."*

---

## Time box

- **Speak:** 4:00–5:00.
- **Q&A:** 3 minutes after.

---

## The Strong-Hire structure

### Beat 1 — Frame the decision (30 sec)
Don't rush to a recommendation. Name the three concerns the room will weigh: **residency + IAM/audit, procurement + commercial, feature velocity**. The right answer must satisfy all three.

### Beat 2 — Walk the three options against the three concerns (2:30)
A 3×3 spoken matrix.

| Surface | Residency + IAM/audit | Procurement | Feature velocity |
|---|---|---|---|
| First-party API | Verify residency for Indian BFSI; not cloud-IAM-native | Adds an Apic contract | Newest features first |
| Bedrock Mumbai | Strong: ap-south-1, IAM-native, CloudTrail | Rolls into AWS spend | Feature lag possible |
| Vertex Mumbai | Strong: asia-south1, IAM-native, Cloud Audit Logs | Rolls into GCP spend | Feature lag possible |

### Beat 3 — Recommend (1:00)
> *"Given 80% AWS, fresh CIO appetite for new vendors, and CISO weight on residency: **Bedrock Mumbai for the API workloads, and Claude for Work for the end-user productivity layer**. The reasons — IAM-native against existing CloudTrail and KMS, residency anchored, procurement clean. The trade-off I'd surface explicitly is feature lag — newest Claude features land first-party first. We mitigate by designing the eval framework to accept model substitution, which means new capabilities can be validated and rolled in within a sprint when they reach Bedrock."*

### Beat 4 — Address the CISO + CIO concerns explicitly (1:00)
- **CISO:** *"CloudTrail-native logging, KMS for encryption, IAM-native access control. We won't introduce a new logging pipeline you have to audit separately."*
- **CIO:** *"Procurement is one Bedrock line-item rather than a new vendor contract. Roadmap velocity gap is real but bounded — we'll show you the substitution path when newer capabilities ship."*

---

## Rubric

### Strong Hire
- Decision framed before recommended (don't rush).
- 3×3 matrix audible (3 surfaces × 3 concerns).
- Recommendation specific (surface + product layer combination), not generic.
- Trade-off named and *mitigated*, not hidden.
- CISO and CIO addressed in their own concerns, separately.
- 4:00–5:00 wall clock.

### Hire
- Recommendation lands but rationale is generic ("Bedrock is what most banks use").
- One concern (CISO or CIO) under-addressed.
- Feature-lag trade-off named without mitigation.

### Lean No
- "It depends." (No recommendation.)
- Generic vendor comparison without BFSI-specific framing.
- Skipping the procurement axis.

### Strong No
- Recommending something the customer can't actually do (e.g. residency-incompatible).
- "Apic is the safest cloud" — wrong axis.
- Disparaging a hyperscaler.

---

## Q&A defense — likely follow-ups

### Q — *"What if our GCP team pushes back?"*
A — Open the door for the data team to use Vertex Mumbai for *their* workloads — multi-surface Claude is fine if eval framework is consistent. The decision isn't AWS-or-GCP company-wide; it's per-workload.

### Q — *"How do we avoid Apic vendor lock-in?"*
A — Three layers of substitutability. Same answer as discovery objection 3 in Module 03. Reference [Module 03 Drill 03 — Three Discovery Objections](../../03-Customer-Discovery-and-Pre-Sales/Drills/03-Three-Discovery-Objections.md).

### Q — *"When would you recommend first-party API instead?"*
A — Customer is multi-cloud / cloud-agnostic and wants the newest capabilities (extended caching, just-shipped tool features) for a flagship use case. Less common in mature BFSI; more common in Tier-2 / new-AI buyers.

### Q — *"How does Claude for Work fit in if we're using Bedrock?"*
A — Different layer. Bedrock is the API/integration layer; Claude for Work is the end-user productivity layer (Projects, Desktop). You can run both — most mature BFSI customers do.

---

## Common Lean No traps

### Trap 1 — Cloud loyalty / rivalry
"AWS is better than GCP." Strong No. Not the question.

### Trap 2 — Single recommendation, no trade-off
"Bedrock Mumbai." Period. Lean No.

### Trap 3 — Skipping Claude for Work
The CIO's "end-user productivity" concern goes unaddressed.

### Trap 4 — Marketing-speak feature velocity
"Apic ships fast." Without the *substitution-path* answer, this is just a hedge.

### Trap 5 — Ignoring the CISO
Speaking to the CIO and not the CISO. The CISO is the most likely veto.

---

## How to run this drill

1. Cold record. Speak the 5-min answer.
2. Have a partner play CIO + CISO and run 3 Q&A follow-ups.
3. Score against rubric.
4. Re-run with a variant (GCP-shop instead, or first-party-leaning new buyer).
5. Application trigger: 2 Strong Hires across 2 variants.

---

## Cross-references
- Note: [05 — Deployment Surfaces](../Notes/05-Deployment-Surfaces.md).
- Module 03: [Three Discovery Objections](../../03-Customer-Discovery-and-Pre-Sales/Drills/03-Three-Discovery-Objections.md).
- Module 09: [PII / DPDPA / RBI Posture](../../09-Security-Privacy-Governance/Notes/03-PII-DPDPA-RBI-Posture.md).
- [Drill Tracker](../../Drill-Tracker.md).

## Strong-Hire bar for this drill
- 3×3 matrix audible without reaching for slides.
- Recommendation specific (Bedrock Mumbai + Claude for Work).
- CISO and CIO addressed in their own languages.
- Feature-lag trade-off mitigated, not hidden.
