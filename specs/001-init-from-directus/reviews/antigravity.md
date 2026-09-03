# SpecKit Review: 001-init-from-directus

**Reviewer**: antigravity
**Reviewed at**: 2026-07-21T10:37:11Z
**Commit**: ae4fbca508498ddcecfb8f73df3c100ee56103da (repos/undrlla)
**Artifacts reviewed**: spec.md, plan.md, tasks.md, data-model.md, research.md, quickstart.md, contracts/openapi.yaml, .specify/memory/constitution.md. Cross-read prior reviews: analyze.md, claude.md, codex.md, copilot.md.

## Summary

The feature specification and data model for `001-init-from-directus` are robust and detailed. However, two critical vulnerabilities prevent implementation readiness: the OpenAPI storefront contract does not include payment instructions (address/amount/quote) for crypto checkouts, rendering crypto purchases unpayable from the published API; and the collect-and-forward crypto settlement design creates unaddressed custodial, MSB, and MiCA regulatory liabilities. Additionally, authorization gaps on guest orders and missing merge rules for anonymous carts create high security risks.

## Findings

| ID | Severity | Area | Finding | Recommendation |
|---|---|---|---|---|
| F1 | CRITICAL | Contract / Payments | `contracts/openapi.yaml:86-104` `/store/checkout` returns a `CheckoutGroup` with `PaymentAttempt` records, but `PaymentAttempt` (`openapi.yaml:418-429`) only exposes `payout_wallet_snapshot` (the vendor's payout address). The customer's payment instruction object (deposit address, exact crypto amount, quote expiration, memo) is completely absent. Storefront and Mini App clients cannot present payment details to complete a crypto checkout. | Add a `payment_instructions` schema to `PaymentAttempt` returning deposit address, currency, exact amount, quote expiration, and memo/tag. |
| F2 | CRITICAL | Security / Compliance | `research.md:57-65` (D7) selects a collect-and-forward model for TON and SHKeeper where customer funds land in a platform-controlled settlement wallet before vendor payout. Holding user crypto funds constitutes financial custody under US FinCEN (MSB) and EU MiCA (CASP) regulations. The spec, runbook, and threat model do not address license requirements, AML reporting, or custody risks. | Re-evaluate direct-to-vendor payment rails or explicitly document legal custody requirements, AML compliance obligations, and geographic restriction gating. |
| F3 | HIGH | AuthZ / Contract | `contracts/openapi.yaml:15-17` sets global security `[{directusAuth: []}, {anonCart: []}]`. Public endpoints like `/items/products` inherit this requirement, blocking unauthenticated storefront visitors from browsing catalog items. No `/store/anon-token` bootstrap endpoint is declared in OpenAPI. | Override security to `[]` on public GET endpoints and document the `anon_token` issuance and cookie-set handshake. |
| F4 | HIGH | Security / Data Model | Guest orders are retrieved via `/store/checkout/{checkout_group_id}` (`openapi.yaml:106-117`). `Order` and `OrderSplit` (`data-model.md:62-71`) do not record `anon_token` or guest ownership hashes, relying solely on path parameter knowledge. Guessable or leaked checkout URLs expose full order details and PII in violation of FR-034. | Include `anon_token` / owner hash on `Order` and require token validation on guest order reads, or issue a non-enumerable `checkout_secret`. |
| F5 | HIGH | Spec Gap / Logic | FR-027 (`spec.md:198`) specifies merging anonymous carts into user accounts upon authentication, but omits conflict resolution rules when both carts contain the same `product_id` and `variant_id`. | Specify deterministic merge logic (summing quantities up to stock caps) and add explicit test cases for concurrent logins. |
| F6 | HIGH | State Machine | Booking rejections and overpayments (`spec.md:201,203`) trigger system auto-refunds via `RefundRequest` (`data-model.md:78`), but `RefundRequest` requires human `approved_by` fields with no `initiator: system` state or bypass path. | Add `initiator: system | customer` to `RefundRequest` and define automated approval transitions for system-triggered refunds. |
| F7 | HIGH | Effort Estimation | Task U-T9 (`tasks.md:59-62`) allocates 2 dev-days for dual-mode ACME TLS, `undevops` pipeline templates, secret injection/rotation, and runbook documentation. Realistically this scope requires 5-8 dev-days. | Adjust U-T9 estimate to 5 dev-days minimum and split into distinct pipeline and secret-management tasks. |

## Alternative approaches considered

- **Direct-to-Vendor Crypto Payments**: Direct transfer to vendor wallet eliminates platform custody and licensing scope, though it complicates fee collection and automated refunds.
- **Guest Checkout Authorization**: Issuing a single-use `checkout_secret` at checkout instead of linking `anon_token` to order records provides non-enumerable order access without schema overhead.

## VERDICT

```yaml
verdict: CRITICAL
reviewer: antigravity
reviewed_at: 2026-07-21T10:37:11Z
commit: ae4fbca508498ddcecfb8f73df3c100ee56103da
critical_count: 2
high_count: 5
medium_count: 0
low_count: 0
```
