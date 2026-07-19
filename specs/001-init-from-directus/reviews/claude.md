# SpecKit Review: 001-init-from-directus

**Reviewer**: claude
**Reviewed at**: 2026-07-19T00:00:00Z
**Commit**: 29ff0d00932b20ade5f1809ef2a964c115f18c04 (repos/undrlla)
**Artifacts reviewed**: spec.md, plan.md, tasks.md, constitution.md. NOT FOUND (skipped): research.md, data-model.md, quickstart.md, contracts/.

## Summary
The spec is unusually thorough and internally well-clarified (12 clarification Q&A, 27 FRs, 13 success criteria with strong idempotency/concurrency coverage). However tasks.md does not fully trace back to the in-scope requirements: the v1 service-booking flow has no implementation task, several in-scope FRs (refunds, delivery zones, digital entitlements) lack dedicated tasks, and two tasks implement Tier-B modules that the spec explicitly declares out of scope. Design-phase artifacts promised by plan.md (research/data-model/quickstart/contracts) are absent.

## Findings
| ID | Severity | Area | Finding | Recommendation |
|---|---|---|---|---|
| F1 | HIGH | A (traceability) | Service booking is in-scope v1 (FR-019 spec:182, SC-008 spec:230, SC-009 spec:231) but no task implements it. `Booking` appears only as a schema deliverable in U-T1 (tasks:11); no task covers slot selection, confirm/reject, overlapping-request rejection, or reschedule/cancel logic. | Add an explicit `[BE]` booking/service-checkout task (confirm-atomicity + concurrency) and wire SC-008/SC-009 e2e into U-T11. |
| F2 | HIGH | A/I (scope) | Spec repeatedly declares Tier-B (Mutual Aid, VPN, Taxi, Barter) OUT OF SCOPE for 001 (spec:40, spec:169, FR-016 spec:179). Yet U-T13 (tasks:75-79, 4d) and U-T14 (tasks:81-85, 4d) implement those collections/UI, and U-T15 depends on U-T13. This is 8d of out-of-scope work contradicting the stated boundary. | Remove/defer U-T13/U-T14 to `002+`, or reclassify U-T13 strictly as the FR-016 extension-point scaffolding (no flagship collections). Re-point U-T15 to Tier-A. |
| F3 | HIGH | A (missing artifacts) | plan.md Deliverables (plan:44) and Phase 0 (plan:26) promise research.md, data-model.md, quickstart.md; none exist. No `contracts/` dir exists, yet FR-014 (spec:177) and SC-006 (spec:228) require a "versioned OpenAPI/GraphQL contract" that storefront + Mini App share and U-T11 contract tests validate. | Produce data-model.md and a contract stub (OpenAPI/GraphQL) before U-T9/U-T10/U-T11 begin; these are prerequisites, not optional. |
| F4 | MEDIUM | A (traceability) | In-scope FR-020 refunds (spec:183/SC-010), FR-021 delivery zones (spec:184/SC-011), and FR-018 digital entitlement/download (spec:181/SC-007) have no dedicated task; U-T6 covers only cart/split/inventory (tasks:39). Coverage is implied but unassigned. | Add tasks or explicitly fold these into U-T6 deliverables so ownership and estimates exist. |
| F5 | MEDIUM | E (security) | Multi-vendor cart (FR-027 spec:190) holds CartItems across multiple `tenant_id`s owned by one customer, but FR-003 (spec:165) enforces per-`tenant_id` row filters. How a single customer cart/order-split reads across tenant-scoped rows under Directus policies is unspecified — a plausible RLS gap or over-restriction. | Specify the row-filter model for customer-scoped cross-tenant reads; add a cross-tenant leak + cart-visibility case to U-T15. |
| F6 | MEDIUM | D (failure modes) | Webhook processing has dedup/idempotency (FR-013 spec:176) but no timeout/retry/circuit-breaker, and no handling for missing webhooks or crypto under/overpayment beyond `quote_expires_at`. Notification adapter (FR-026) has email fallback but no retry/backoff spec. | Define webhook retry/reconciliation policy and crypto amount-mismatch states; specify notification delivery retry semantics. |
| F7 | MEDIUM | E (security/privacy) | Guest `anon_token` cart (spec:202) and `contact_handle` guest notifications (FR-027 spec:190) lack token entropy/expiry, hijack protection, and abuse/enumeration controls (arbitrary handle → notification spam). | Specify token generation/expiry and rate-limiting/verification for guest contact handles. |
| F8 | MEDIUM | E (secrets) | Per-instance Stripe keys, webhook signing secrets, and TON/SHKeeper credentials are central to FR-013/FR-025 but no artifact states where/how they are stored or injected during `undevops` provisioning (U-T8). | Add secret-handling to the provisioning runbook (U-T8) and reference in FR-022/FR-023. |
| F9 | LOW | A (estimates) | plan.md estimates "23-24 developer-days" (plan:40) but tasks.md dispatch totals 43d (tasks:123-128), the delta being flagship U-T13/U-T14 + security U-T15. | Reconcile estimate with task total after resolving F2. |
| F10 | LOW | I (constitution) | The constitution (`.specify/memory/constitution.md`) governs the `clai-helpers` CLI and `.claude/` template pipeline — a different domain than this Directus marketplace feature. Its principles (Source-of-Truth, Transformer-not-Fork, SemVer) do not map onto these artifacts. No violations found; alignment is effectively N/A. | Note the domain mismatch; confirm whether a feature-appropriate constitution should govern `repos/undrlla`. |

## Alternative approaches considered
- Multi-tenancy (lens G): The spec chose instance-per-client isolation plus in-instance row filters (Clarification spec:39). A shared-instance `tenant_id`-only model was explicitly weighed and rejected for hard isolation — this is documented and reasonable; no gap.
- Payments: A single-provider v1 with later expansion was an available alternative; spec deliberately makes all three first-class behind one abstraction (spec:42). Trade-off (more surface, more test burden) is accepted but raises the F6 reconciliation risk.

## VERDICT
```yaml
verdict: HIGH
reviewer: claude
reviewed_at: 2026-07-19T00:00:00Z
commit: 29ff0d00932b20ade5f1809ef2a964c115f18c04
critical_count: 0
high_count: 3
medium_count: 4
low_count: 2
```
