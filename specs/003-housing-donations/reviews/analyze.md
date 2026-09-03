# SpecKit Analyze: 003-housing-donations

**Reviewer**: analyze (author self-consistency audit — NOT an external review under Constitution Principle VI)
**Reviewed at**: 2026-07-21
**Commit**: (to be filled at commit time)
**Artifacts**: spec.md, data-model.md, research.md, quickstart.md
**Status**: Pre-implementation. No `plan.md` / `tasks.md` / `contracts/` yet — those are produced AFTER this analyze PASS and after ≥2 external reviewer PASS verdicts (per spec FR-013 and Constitution Principle VI).

> **Important**: this file is the **analyze** stage of the Constitution VI gate (Gate 1). It is authored by the spec author as a self-consistency audit. It is NOT one of the two required external-AI reviewer files. The author of a spec cannot be an external reviewer of it. External reviewers write `reviews/<provider>.md` (e.g. `reviews/codex.md`, `reviews/claude.md`, `reviews/copilot.md`, `reviews/gemini.md`) in separate sessions.

## Findings

| ID  | Category        | Severity | Location(s)                                       | Summary                                                                                                                | Recommendation                                                                                                            |
| --- | --------------- | -------- | ------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| A1  | Scope boundary  | LOW      | spec.md FR-008, data-model.md "Concierge services" | Concierge services reuse `001` `Service`/`Booking` unchanged; confirmed zero new collection.                            | Acceptable; flagged for external reviewers to verify no hidden coupling.                                                  |
| A2  | Open dependency | MEDIUM   | spec.md Executive Context, research.md D4         | `HousingPool` relationship to a future MAQ is deferred to the MAQ spec author.                                          | Acceptable; non-blocking. External reviewers should confirm `003` is genuinely implementable without MAQ existing.        |
| A3  | Legal dependency| MEDIUM   | spec.md FR-012, research.md "Operational risks"   | KYC/AML provider integration is out of scope; only the `kyc_status` flag existence is required.                          | Acceptable for MVP. External reviewers should flag if this creates an unimplementable transition (it does not — flag-only).|
| A4  | Schema risk     | LOW      | data-model.md "Migration / extension packaging"  | `001` collections referenced read-only by `003` (Service, Booking, PaymentAttempt, Order, OrderSplit); no `001` change. | Acceptable; verify at tasks phase that no `001` migration is silently required.                                            |
| A5  | Ambiguity       | LOW      | spec.md FR-008, data-model.md `ServiceArea` link  | "MAY be linked to one or more `ServiceArea` records" — the M2M join collection shape is not fully specified.             | Defer concrete M2M shape to tasks phase; spec intent is clear (refuse outside-area bookings).                              |
| A6  | Operational risk| LOW      | research.md "Operational risks"                   | Long TRY escrow holds + FX volatility; MVP records `settlement_rate_snapshot` but does not re-peg.                       | Acceptable for MVP; operational runbook (tasks phase) should minimize hold duration.                                       |
| A7  | Coverage gap    | LOW      | spec.md (no FR for notification events)           | Spec does not enumerate housing-specific notification events (application submitted/verified/approved/funded/released).   | Add to tasks phase: extend the `001` FR-026 event catalog with housing events. Not a spec defect.                          |
| A8  | Ambiguity       | LOW      | spec.md FR-002 "complete its current lifecycle stage" | "allowed to complete its current stage but MUST NOT advance" — exact terminal-safe states not enumerated.            | Clarify in tasks phase: any non-terminal status parks; `completed` / `rejected` stay terminal.                             |

No CRITICAL or HIGH findings. All MEDIUM findings are conscious scope/dependency decisions documented in `research.md`, not defects.

## Coverage Summary

Mapping each spec FR to its planned implementation locus. Task IDs are provisional (formal `tasks.md` is produced post-review).

| Requirement Key                | Has Task Locus? | Locus (provisional)                    | Notes                                                                                |
| ------------------------------ | --------------- | -------------------------------------- | ------------------------------------------------------------------------------------ |
| FR-001 (ServiceArea)           | Yes             | BE: schema migration (extension pkg)   | Universal collection; admin UI via Directus Studio.                                  |
| FR-002 (Intake gating)         | Yes             | BE: extension validation hook          | Reject at intake; park mid-application.                                              |
| FR-003 (HousingProgram)        | Yes             | BE: schema + admin CRUD                | Active-state guarded by linked ServiceArea.enabled.                                   |
| FR-004 (HousingApplication)    | Yes             | BE: schema + state-machine extension   | Status machine; duplicate detection.                                                 |
| FR-005 (HousingEscrow)         | Yes             | BE: schema + destination-snapshot hook | Mirrors `PaymentAttempt.payout_wallet_snapshot` (`001` FR-025).                       |
| FR-006 (HousingPoolAllocation) | Yes             | BE: schema + atomic debit hook         | MVP: full-amount only.                                                               |
| FR-007 (HousingPool)           | Yes             | BE: schema + credit/debit audit        | Housing-only; MAQ merge deferred (research D4).                                       |
| FR-008 (Concierge via Service) | Yes             | FE+BE: seed 4 `Service` rows           | Zero new collection; reuses `001`.                                                   |
| FR-009 (Model B)               | N/A — Deferred  | (none in `003`)                        | Deferred; SC-010.                                                                    |
| FR-010 (Model C)               | N/A — Deferred  | (none in `003`)                        | Deferred; SC-011.                                                                    |
| FR-011 (PII handling)          | Yes             | BE: FLS policies + audit redaction     | Extends `001` FR-036.                                                                |
| FR-012 (KYC/AML gating)        | Yes (flag-only) | BE: `kyc_status` fields + transition guard | Provider integration out of scope (research).                                    |
| FR-013 (Cross-AI Review Gate)  | This file + ext | Process                                | Gate 1 = this analyze; Gate 2 = ≥2 external reviewer files.                          |
| FR-014 (Isolated extension)    | Yes             | BE+OPS: extension package + registration | Clean Tier-A instance unaffected (SC-008).                                         |
| FR-015 (Admin UI)              | Yes             | Directus Studio (MVP); custom deferred | Same row filters / FLS as API.                                                       |
| FR-016 (Applicant + concierge clients) | Yes     | FE: storefront + Mini App contract delta | Contract delta defined in tasks phase.                                        |

## Constitution Alignment Issues

- **Principle VI (Cross-AI Review Gate)**: Gate 1 (`analyze`) status is **PASS** (this file). Gate 2 (≥2 distinct external-AI reviewer files in `reviews/<provider>.md` with verdict PASS) is **NOT YET SATISFIED** — must be completed before any implementation task begins. The author of this spec MUST NOT be one of those external reviewers.
- **Principle VII (Artifact Versioning)**: when the implementation branch is created and `plan.md` / `tasks.md` are produced, each stage MUST be tagged via `.specify/scripts/{bash,powershell}/snapshot-stage.{sh,ps1}` using `<stage>/003-housing-donations/v<N>`. No parallel `.history/` files.
- **Principle IX (Two-Phase Review Flow)**: this spec lives on the `specs/003-housing-donations` branch (planning phase). The implementation branch (`003-housing-donations` from `main`) is created only after the planning PR merges and Gate 2 is satisfied.

## Unmapped Tasks

N/A — `tasks.md` is not yet produced. It will be generated after Gate 2 PASS, via the SpecKit tasks stage (manual in this fork; no slash-commands present).

## Metrics

- Total Functional Requirements: 16 (of which 2 are explicitly Deferred: FR-009, FR-010)
- Total Success Criteria: 11 (of which 2 are explicitly Deferred: SC-010, SC-011)
- Coverage % (non-deferred FRs with a task locus): 100% (14/14)
- Ambiguity count: 3 (A5, A7, A8 — all LOW, all resolvable in tasks phase)
- Duplication count: 0
- CRITICAL count: 0
- HIGH count: 0
- MEDIUM count: 2 (A2, A3 — both conscious scope/dependency decisions documented in research.md)
- LOW count: 6

## Open questions for external reviewers

External reviewers should specifically evaluate:

1. **MAQ independence (A2)**: is `003` genuinely implementable without a future MAQ spec existing? Is the `HousingPool` design reversible if MAQ later merges?
2. **Concierge reuse soundness (A1)**: does reusing `001` `Service`/`Booking` for concierge create any hidden coupling or gap (e.g. notifications, refunds, geo-scoping) that a dedicated collection would not?
3. **KYC gating implementability (A3)**: is "flag-only" KYC sufficient for MVP, or does it create a transition that cannot be operationally satisfied?
4. **Escrow snapshot adequacy (FR-005)**: is the `destination_account_ref` snapshot semantics (mirroring `PaymentAttempt.payout_wallet_snapshot`) sufficient for property transactions where signing may be weeks away?
5. **ServiceArea universal design (FR-001)**: is the `feature_scope` enum approach sound for future Tier-B features (delivery, taxi, barter), or does it need a different shape?
6. **PII boundary (FR-011)**: is the vendor-reads-only-what-is-needed rule enforceable via Directus field permissions alone, or does it require row-level computation?

## VERDICT

```yaml
verdict: PASS
reviewer: analyze
reviewed_at: 2026-07-21
commit: TBD-at-commit
critical_count: 0
high_count: 0
medium_count: 2
low_count: 6
note: >
  Self-consistency audit by the spec author. Constitution Principle VI Gate 1
  satisfied. Gate 2 (≥2 distinct external-AI reviewer PASS verdicts in
  reviews/<provider>.md) is NOT yet satisfied and MUST complete before any
  implementation task begins.
```
