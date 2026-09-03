# SpecKit Review: 003-housing-donations

**Reviewer**: antigravity
**Reviewed at**: 2026-07-21T10:37:11Z
**Commit**: ae4fbca508498ddcecfb8f73df3c100ee56103da (repos/undrlla)
**Artifacts reviewed**: spec.md, data-model.md, research.md, quickstart.md, reviews/analyze.md. NOT FOUND (skipped): plan.md, tasks.md, contracts/ delta.

## Summary

`003-housing-donations` presents an exceptionally well-designed domain spec, complete with a clean `ServiceArea` feature-gating model, an immutable destination-snapshot escrow pattern, and zero-schema-bloat reuse of `001` `Service`/`Booking` collections for concierge features. However, implementation cannot proceed because `plan.md` and `tasks.md` are missing (blocking Constitution Principle VI). Additionally, long-duration escrow holds in volatile fiat currencies (TRY) and flag-only KYC gating pose real financial and operational compliance risks.

## Findings

| ID | Severity | Area | Finding | Recommendation |
|---|---|---|---|---|
| F1 | CRITICAL | SpecKit Pipeline | Core planning artifacts `plan.md` and `tasks.md` are absent. Under Constitution Principle VI and Principle IX, `/speckit.implement` is blocked until plan and tasks artifacts are completed and approved. | Generate `plan.md` and `tasks.md` via the SpecKit pipeline before proceeding to implementation. |
| F2 | HIGH | Compliance / Legal Risk | `spec.md:152` (FR-012) requires `kyc_status` flags on `HousingApplication` and `HousingEscrow`, but explicitly defers real KYC/AML provider integration as an out-of-scope legal dependency (`research.md` D4). Real estate transactions in Turkey and Georgia carry strict statutory AML thresholds. Operating with unverified flag toggles creates immediate regulatory liability. | Define a concrete operational requirement or integration interface for KYC/AML verification prior to enabling live escrow payouts. |
| F3 | HIGH | Financial / FX Risk | `data-model.md:52` records `settlement_rate_snapshot` on `HousingEscrow` at creation time, but omits FX re-pegging or currency hedging mechanisms for long-duration escrow holds in volatile currencies like Turkish Lira (TRY). Escrow holds spanning weeks risk severe currency devaluation against target property prices. | Implement configurable re-pegging thresholds or mandate hard currency (EUR/USD/USDT) settlement for `HousingEscrow` balances. |
| F4 | HIGH | Security / PII Protection | FR-011 (`spec.md:155`) mandates PII protection for property addresses, identity docs, and notary IBANs. While FLS policies are described in `data-model.md:18-28`, redacting PII from `NotificationDelivery.last_error` logs and automated retention purging require explicit tasks in the upcoming plan. | Include automated PII log sanitization middleware and retention cleanup cron jobs in the implementation plan. |
| F5 | MEDIUM | Data Model / Architecture | `data-model.md:60` specifies `HousingPool` balance debits and credits, but does not define how incoming donations recorded via `001` `PaymentAttempt` / `Order` map into `HousingPool` credits. | Define the Directus event hook or trigger linking completed donation `Order` line items to `HousingPool.balance` credits. |

## Alternative approaches considered

- **Hard-Currency Escrows**: Restricting `HousingEscrow` amounts strictly to stablecoins (USDT) or EUR/USD mitigates TRY volatility risks without requiring complex FX hedging engines.
- **Dedicated Concierge Collections**: Spec author correctly rejected this approach (D3), opting to reuse `001` `Service` and `Booking` models. This decision is sound and avoids schema duplication.

## VERDICT

```yaml
verdict: CRITICAL
reviewer: antigravity
reviewed_at: 2026-07-21T10:37:11Z
commit: ae4fbca508498ddcecfb8f73df3c100ee56103da
critical_count: 1
high_count: 3
medium_count: 1
low_count: 0
```
