# SpecKit Review: 003-housing-donations

**Reviewer**: grok
**Reviewed at**: '"$ts"'
**Commit**: '"$c001"'
**Artifacts reviewed**: spec.md, plan.md, tasks.md, data-model.md, research.md, contracts/openapi.yaml, FLOWS-AUDIT.md; constitution VI

## Summary

Housing+CBI pot + queue **model B** is now product-coherent (shared pot, 5% fee, active sub keeps seat, FIFO with pause/90d). Escrow-never-cash-to-applicant is strong. Remaining cracks: **subscription product not fully modeled**, chargeback/ledger reverse paths, queue head-of-line blocking, KYC provider still open, and FR numbering collision (FR-020 donation fee vs ecosystem mental model of 001 FR-020 refunds — different files but confusing).

## Findings

| ID | Severity | Area | Finding | Recommendation |
|---|---|---|---|---|
| F1 | HIGH | Data model / FR gap | Subscription lifecycle (create/cancel/past_due, Stripe Billing vs custom) is FR-described but **no Subscription entity** in data-model.md; only application queue fields. | Add `HousingPoolSubscription` (+ payment events) to data-model and OpenAPI. |
| F2 | HIGH | Failure modes | Non-refundable donations + chargebacks: FR-023 says reverse ledger — **no algorithm** (which applications if pool already allocated? clawback?). | Define pool accounting: FIFO spend, chargeback hits unallocated balance first; if negative, admin freeze allocations. |
| F3 | HIGH | Edge case | Head of queue with `requested_amount` > balance **blocks** everyone (MVP wait, no skip). Starvation if head wants $500k and inflow is slow. | Document operator playbook; consider optional admin "skip with reason" (already override) and UI transparency of "waiting for pot". |
| F4 | MEDIUM | Security / compliance | KYC/AML required by status fields; provider integration out of scope — **high-risk** for CBI-adjacent marketing. | Block public "passport" marketing until KYC provider + legal review; keep verified gate hard. |
| F5 | MEDIUM | Consistency | Concierge founder-only MVP vs post-MVP self-signup FR-019 — OK, but tasks still "four services founder" only; no task for queue B / subscription. | Expand tasks.md for sub billing, queue worker, fee split on donation Orders. |
| F6 | MEDIUM | Logical consistency | FR-006 full-amount only + model B sub: **progress toward one person** not shown to donors (correct for non-earmark) but applicant dashboard needs "position + estimated wait" — unspecified. | FR: read-only queue position estimate (optional, non-binding). |
| F7 | LOW | Naming | FR-020 donation fee in `003` vs FR-020 refunds in `001` — cross-spec confusion. | Renumber 003 donation fee to FR-024+ free numbers or prefix HD-FR. |
| F8 | CRITICAL | Gate artifacts | Prior analyze/antigravity reviews may predate queue B / 5% / sub rules; **stale review risk**. plan/tasks may lag FR-021–023. | Re-run analyze after product locks; refresh tasks; second listed-provider review required for implement gate. |

## Alternative approaches considered

- Earmarked campaigns (rejected product) — better marketing, worse AML.
- Model A one-time seat (rejected) — simpler, kills pot economics.

## VERDICT

```yaml
verdict: HIGH
reviewer: grok
reviewed_at: '"$ts"'
commit: '"$c001"'
critical_count: 1
high_count: 3
medium_count: 3
low_count: 1
```
