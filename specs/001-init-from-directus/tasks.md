# Tasks: 001-init-from-directus (undrlla)

## Overview

Tasks generated from `plan.md` and `spec.md`. Tasks include explicit agent tags, estimates, formal dependency graphs, and multi-lane execution graphs. Tier-B flagship collections/UI (Mutual Aid, VPN, Taxi, Barter) are OUT OF SCOPE for `001-init` (spec §Clarifications, FR-016) and deferred to a future `002+` milestone; `001` ships only the extension-point scaffolding (U-T13) that Tier-B plugs into later.

## Tasks

1. [BE] U-T1 — Schema & Directus extension scaffolding
   - Owner: `backend-specialist`
   - Est: 3d
   - Deliverables: `schema.snapshot.yaml`, core collection defs (Product, Variant, Service, Booking, Cart, CartItem, Order, OrderSplit, PaymentAttempt, RefundRequest, DeliveryZone, DigitalEntitlement, InventoryReservation, VendorPayoutAccount, TelegramLink, NotificationDelivery, TaxRegistration) with policy row-filter/field-permission rules per `data-model.md`. Encodes the customer-scoped cross-tenant read model (FR-031).

2. [BE] U-T2 — Payment-provider abstraction core
   - Owner: `backend-specialist`
   - Est: 2d
   - Deliverables: provider interface, common `PaymentAttempt` lifecycle, webhook verifier skeleton, async webhook intake + retry/backoff/circuit-breaker + reconciliation-sweep scaffolding (FR-032).

3. [BE] U-T3 — Stripe Connect integration
   - Owner: `backend-specialist`
   - Est: 4d
   - Depends on: U-T2
   - Deliverables: Connect Express onboarding, webhook handlers, application fee handling.

4. [BE] U-T4 — Crypto adapters (TON, SHKeeper)
   - Owner: `backend-specialist`
   - Est: 3d
   - Depends on: U-T2
   - Deliverables: checkout quotes, collect-and-forward settlement through per-instance crypto settlement wallets, vendor payout wallet snapshot on `PaymentAttempt`, platform-fee retention during reconciliation, HMAC signature verification, idempotency replay protection, 200 OK webhook handlers, crypto amount-mismatch state handling (`underpaid`/`overpaid`/`expired`, FR-032).

5. [BE] U-T5 — Tax-provider abstraction & Stripe Tax integration
   - Owner: `backend-specialist`
   - Est: 3d
   - Deliverables: tax-provider interface, Stripe Tax adapter, VIES VAT validation hook.

6. [BE] U-T6 — Cart, multi-vendor checkout, fulfilment, refunds & delivery
   - Owner: `backend-specialist`
   - Est: 4d
   - Depends on: U-T1, U-T2
   - Deliverables:
     - Cart & checkout: cart splitting, `checkout_group_id`, `OrderSplit` fan-out, inventory reservation lock.
   - Mixed checkout outcomes: paid sibling vendor orders remain committed when another vendor order fails/expires/refunds; failed vendor orders release only their own reservations; clients read aggregate `partially_*` checkout-group states.
     - Cross-tenant read model (FR-031): customer-identity-filtered cart/order reads; vendor reads `tenant_id`-scoped only.
     - FR-018 fulfilment: physical inventory reserve/consume/release, manual fulfilment + tracking number, digital `DigitalEntitlement` grant + protected time-limited download link with revocation.
     - FR-020 refunds: `RefundRequest` create → vendor/admin approve/reject → adapter call; approved ≤ captured; only provider-confirmed refunds mutate `PaymentAttempt`/`Order`.
     - FR-021 delivery: `DeliveryZone` matching, fixed/free rates, local pickup, no-match = delivery unavailable (no fallback rate).

7. [BE] U-T7 — Service booking & service checkout
   - Owner: `backend-specialist`
   - Est: 3d
   - Depends on: U-T1, U-T2
   - Deliverables (FR-019, FR-030): slot selection + `pending` booking creation; provider confirm/reject with atomic slot reservation; overlapping-pending-request rejection with concurrency-safe locking (no double booking); reschedule/cancel with slot release; pre-paid booking → linked `PaymentAttempt` (card authorize-on-request/capture-on-confirm/void-on-reject; crypto charge-on-request/auto-refund-on-reject via FR-020 path); pay-later/free bookings create no payment.

8. [BE] U-T8 — Notifications extension
   - Owner: `backend-specialist`
   - Est: 2d
   - Deliverables: Directus events → channel adapter (Telegram primary + email fallback), NotificationDelivery audit log, per-channel retry with bounded exponential backoff and no duplicate-on-success (FR-033).

9. [OPS] U-T9 — Provisioning runbook & undevops templates
   - Owner: `devops-engineer`
   - Est: 2d
   - Deliverables: operator runbook, templated deploy pipeline, ACME domain & SSL flow, and per-instance secret handling (FR-035): generation, deploy-time injection via `undevops` secret store/env (Stripe keys + webhook signing secrets, TON/SHKeeper credentials), reference-only wiring into extensions, rotation and revocation procedures.

10. [FE] U-T10 — Frontend: `undrllanding` Next.js storefront
    - Owner: `frontend-specialist`
    - Est: 5d
    - Depends on: U-T1, U-T6, U-T7, U-T8
    - Deliverables: Next.js storefront with dynamic `@underundre/undesign` theme loading; catalog, cart, checkout, service-request journeys against the versioned contract (`contracts/`).

11. [FE] U-T11 — Telegram Mini App client integration
    - Owner: `frontend-specialist`
    - Est: 2d
    - Depends on: U-T1, U-T7, U-T8
    - Deliverables: TG Mini App view, WebApp auth initData verification, shared contract catalog/cart/checkout/booking surface.

12. [TEST] U-T12 — Contract tests & CI
    - Owner: `test-engineer`
    - Est: 3d
    - Depends on: U-T1, U-T3, U-T4, U-T5, U-T6, U-T7
      - Deliverables: contract tests validating both client-visible `/store/*` OpenAPI and generated Directus REST/GraphQL surfaces (FR-014/SC-006); webhook simulation suite; e2e checkout tests including mixed multi-vendor partial states, tax quote/VIES input, guest cart/contact flow, notification-delivery audit, digital entitlement download access, refund decision/provider-result flow, and pre-paid booking payment policy (SC-012..SC-016); e2e booking tests covering SC-008 (request/confirm/reject/cancel/reschedule reflected in both clients), actor-specific transition authorization, and SC-009 (concurrent overlapping-request confirmation succeeds exactly once, competitors → `rejected`).

13. [BE] U-T13 — Tier-B extension-point scaffolding (FR-016 only)
    - Owner: `backend-specialist`
    - Est: 2d
    - Depends on: U-T1
    - Scope: FR-016 extension-point architecture ONLY — isolated Directus extension package boundaries, migration/registration hook points, and event/plugin seams that future Tier-B specs (`002+`) plug into. NO flagship collections, triggers, or escrow hooks in `001`. A clean Tier-A instance MUST contain none of these modules by default (User Story 1, Acceptance Scenario 2).
    - Deferred to `002+`: `mutual_aid_queue`/`mutual_aid_donations`/`barter_items`/`taxi_requests` collections, Flow-score triggers, escrow payout hooks, and all Tier-B storefront/Mini App UI.

14. [SEC] U-T14 — Security & RLS/FLS Policy Audit
    - Owner: `security-auditor`
    - Est: 2d
    - Depends on: U-T1, U-T6, U-T7
      - Deliverables: multi-tenant cross-tenant leak test, FLS column blacklist audit report, cross-tenant leak + cart-visibility test (customer cross-tenant cart is customer-filtered, vendor cannot read another vendor's split or the full cart per FR-031), guest `anon_token`/`contact_handle` entropy/expiry/abuse checks (FR-034), and PII controls for delivery addresses, VAT/VIES data, Telegram chat IDs, contact handles, and payout wallets: role-specific masks, retention/deletion rule, audit-log redaction, and negative tests that vendors/customers cannot over-read another party's PII (FR-036).

15. [DOC] U-T15 — Quickstart & docs
    - Owner: `documentation-writer`
    - Est: 1d
    - Depends on: U-T9
    - Deliverables: `quickstart.md`, provisioning guides, vendor onboarding docs.

## Dependency Graph

U-T1 → U-T6
U-T1 → U-T7
U-T1 → U-T13
U-T2 → U-T3
U-T2 → U-T4
U-T1 + U-T2 → U-T6
U-T1 + U-T2 → U-T7
U-T1 + U-T6 + U-T7 + U-T8 → U-T10
U-T1 + U-T7 + U-T8 → U-T11
U-T1 + U-T3 + U-T4 + U-T5 + U-T6 + U-T7 → U-T12
U-T1 + U-T6 + U-T7 → U-T14
U-T9 → U-T15

## Parallel Lanes

- Lane A (Core Backend): U-T1 → [U-T6, U-T7] → U-T12
- Lane B (Payments & Tax): U-T2 → [U-T3, U-T4, U-T5] → U-T12
- Lane C (Integration & UI): U-T8 → [U-T10, U-T11]
- Lane D (Extension seams & Security): U-T13 (indep.) ; U-T14 after [U-T6, U-T7]
- Lane E (Provisioning & Docs): U-T9 → U-T15

## Critical Path

U-T1 → U-T6/U-T7 → U-T12 (schema → checkout & booking → contract/e2e tests) is the critical path for validating the Tier-A marketplace core.

## Agent Dispatch Plan

| Owner | Tag | Assigned Tasks | Total Est |
| ----- | --- | -------------- | --------- |
| `backend-specialist` | `[BE]` | U-T1, U-T2, U-T3, U-T4, U-T5, U-T6, U-T7, U-T8, U-T13 | 26d |
| `frontend-specialist` | `[FE]` | U-T10, U-T11 | 7d |
| `devops-engineer` | `[OPS]` | U-T9 | 2d |
| `test-engineer` | `[TEST]` | U-T12 | 3d |
| `security-auditor` | `[SEC]` | U-T14 | 2d |
| `documentation-writer` | `[DOC]` | U-T15 | 1d |

## Total task count: 15

## Total estimate: 41 dev-days (Tier-A `001` scope; Tier-B flagship collections/UI deferred to `002+`)

## Suggested MVP Scope

- Deliver Directus schema (Tier-A), payment-provider adapters for Stripe and crypto rails, multi-vendor cart & checkout with fulfilment/refunds/delivery, service booking, Telegram notifications, and the Next.js / TG Mini App storefront. Tier-B flagship modules are deferred to `002+` and only their extension seams (U-T13) ship in `001`.
