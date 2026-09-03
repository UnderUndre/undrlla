# Phase 0 Research: 001-init-from-directus (undrlla-core)

**Spec**: `spec.md` | **Plan**: `plan.md` | **Date**: 2026-07-18

Brief record of the key architectural decisions taken for the Tier-A rebase, with rationale and rejected alternatives. These mirror the trade-offs already weighed in the spec Clarifications.

## D1 — Multi-tenancy isolation: instance-per-client + in-instance policy row-filters

**Decision**: Hybrid. Each client marketplace is its own isolated Directus instance/container (deployed via `undevops`); within an instance, Directus policy/permission row filters + field permissions separate vendors/customers/sub-orgs via `$CURRENT_USER` (`tenant_id`, `owner_id`).

**Rationale**: Hard cross-client isolation (data, compute, blast radius, per-instance secrets/currency) without depending on `tenant_id` filtering correctness for the security-critical boundary. `tenant_id` handles the softer within-instance vendor separation where Directus policies are expressive enough.

**Alternatives rejected**:

- *Single shared instance, `tenant_id` row-filters only*: one policy bug = full cross-client leak; noisy-neighbor and per-client currency/secret config become awkward.
- *DB-per-tenant in one app*: connection/migration sprawl without the container-level isolation benefits.

**Consequence / open point (FR-031)**: the customer-owned multi-vendor cart deliberately crosses `tenant_id`. Resolved by filtering `Cart`/`CartItem`/customer order-view by **customer identity**, and fanning writes out into `tenant_id`-scoped `Order`/`OrderSplit` per vendor so vendor reads stay `tenant_id`-filtered.

## D2 — Multi-provider payment abstraction (Stripe Connect + TON + SHKeeper)

**Decision**: One common payment-provider interface (uniform checkout/webhook/refund) with a shared `PaymentAttempt` state machine; three first-class v1 adapters behind it.

**Rationale**: Keeps the marketplace core provider-agnostic and each rail independently testable; a single idempotency/dedup/state-transition model covers all providers. Crypto amount-mismatch (`underpaid`/`overpaid`/`expired`) and quote-TTL live on `PaymentAttempt` rather than leaking into checkout logic.

**Alternatives rejected**:

- *Merchant-of-Record (Paddle/Lemon Squeezy)*: MoR becomes the sole seller — incompatible with multi-vendor Stripe Connect payouts. Out of scope.
- *Stripe-only v1*: excludes the crypto rails that are core to the platform's positioning.

**Resilience (FR-032)**: async webhook intake, retry with bounded backoff, per-provider circuit breaker + dead-letter/reconciliation queue, and a reconciliation sweep for stuck non-terminal attempts.

## D3 — Tax abstraction (Stripe Tax + pluggable crypto adapter + VIES)

**Decision**: Tax-provider abstraction mirroring payments; Stripe Tax for card orders, a rates-adapter for crypto/non-Stripe, official VIES for EU B2B reverse-charge, nexus/OSS-gated collection.

**Rationale**: Stripe Tax cannot price crypto rails; the adapter keeps a uniform `Order` tax record (`tax_amount`, `tax_mode`, `reverse_charge`, per-line breakdown) across rails.

## D4 — Secret storage & injection (FR-035)

**Decision**: Per-instance secrets (Stripe keys + webhook signing secrets, TON/SHKeeper credentials) are generated per instance, injected at deploy time via the `undevops` secret store/environment, referenced by extensions via env/secret references only, never committed to `schema.snapshot.yaml` or images, and rotatable without code change.

**Rationale**: Instance-per-client isolation (D1) extends to secrets; keeps PCI/credential blast radius per client and enables rotation/revocation in the provisioning runbook.

## D5 — Guest identity & anti-abuse (FR-034)

**Decision**: `anon_token` = >=128-bit CSPRNG opaque secret in an `httpOnly`/`Secure`/`SameSite` cookie with idle+absolute expiry, rotated on account merge; guest `contact_handle` rate-limited and verified (Telegram deep-link / email confirmation) before high-volume notifications; no enumeration via response/timing differences.

**Rationale**: Prevents cart hijack, notification spam, and handle enumeration on the guest-checkout path.

## D6 — Storefront contract sharing

**Decision**: A single versioned OpenAPI/GraphQL contract (`contracts/`) exported by the platform is consumed by both the `undrllanding` Next.js storefront and the Telegram Mini App (FR-014/SC-006), validated by U-T12 contract tests.

**Contract boundary**: `contracts/openapi.yaml` is authoritative for client-visible `/store/*` workflows (catalog, cart, checkout status, booking transitions, refunds, tax quote, digital entitlement download, notifications, vendor payout setup). Directus generated REST/GraphQL remains authoritative for admin/internal collection CRUD and is validated separately by contract tests.

## D7 — Crypto settlement model for TON / SHKeeper & Compliance Framework

**Decision**: v1 supports both **collect-and-forward** (default for licensed operators) and **direct-to-vendor** settlement for TON and SHKeeper. The settlement mode is configured per instance via `PaymentAttempt.settlement_model` (`stripe_connect_application_fee` | `crypto_collect_and_forward` | `direct_to_vendor`).

**Compliance & Regulatory Note**:
Holding customer crypto funds under collect-and-forward constitutes money transmission and custody under US FinCEN (MSB registration) and EU MiCA (CASP licensing). Deployments operating without an active custodial license MUST set `settlement_model = direct_to_vendor` during instance provisioning (FR-025/FR-035).

**Settlement Mechanics**:
- Under `crypto_collect_and_forward`: Customer payments land in a per-instance platform-controlled settlement wallet. During reconciliation, the platform retains `Order.platform_fee_amount` and queues the vendor payout to the vendor's snapshotted payout wallet.
- Under `direct_to_vendor`: Customer payments land directly in the vendor's snapshotted wallet; platform fees are billed post-settlement or recorded as a vendor account debit.

**Operational rule**: the vendor payout wallet is snapshotted on `PaymentAttempt` at creation. If the vendor changes their wallet later, existing orders/refunds keep the original snapshot; only new payment attempts use the new wallet.

**Customer Payment Instructions**:
Every `PaymentAttempt` returns a structured `payment_instructions` object containing the deposit address, exact crypto amount, quote expiration, and memo/tag so clients can complete checkouts reliably.

## D8 — Partial multi-vendor checkout semantics

**Decision**: vendor orders in a `checkout_group_id` are independent after fan-out. A paid vendor order remains committed when a sibling vendor order fails, expires, or is refunded. The checkout group exposes aggregate mixed states (`partially_paid`, `partially_failed`, `partially_refunded`) to clients.

**Rationale**: rolling back successful vendor orders would create avoidable refund/fulfilment churn and conflicts with FR-024's independent pay/fulfil/refund requirement. Per-vendor state keeps inventory and payment recovery bounded to the affected vendor.

## Operational inputs

- Existing `undevops` deploy runbook and ACME flow for per-domain TLS issuance are reused; both client-owned-domain (ACME per-domain) and platform-subdomain (wildcard TLS) paths are supported (FR-023).
