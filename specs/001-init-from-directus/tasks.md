# Tasks: 001-init-from-directus (undrlla)

## Overview

Tasks generated from `plan.md` and `spec.md`. Tasks include explicit agent tags, estimates, formal dependency graphs, and multi-lane execution graphs. Tier-B flagship collections/UI (Mutual Aid, VPN, Taxi, Barter) are OUT OF SCOPE for `001-init` (spec §Clarifications, FR-016) and deferred to a future `002+` milestone; `001` ships only the extension-point scaffolding (U-T13) that Tier-B plugs into later.

> ## ⛔ MACHINE RULE — client commerce (2026-07-30)
>
> **`status: do-not-implement` for client-shop paths.** Do not execute tasks that build a Directus multi-vendor shop, Directus cart/checkout as client SoT, or undrllanding against Directus store API.
>
> | Concern | Implement instead |
> |---------|-------------------|
> | Client shop engine | `undreseller/specs/001-template-shop` |
> | Storefront | `undrllanding` Medusa Store API |
> | One-click deploy | `undevops/003` + Medusa manifest (`002`) |
> | Crypto | `undrepay` |
>
> **Still candidate later (flagship polity only):** notification patterns, Tier-B extension seams (U-T13), housing-adjacent reuse via `003` — not client marketplace SoT.

## Tasks

1. [BE] U-T1 — Schema & Directus extension scaffolding  
   - **`status: do-not-implement` (client commerce)** — Product/Cart/Order/… as client shop SoT is superseded by Medusa. Do not implement for client shops. Flagship-only residual collections only if a future flagship polity plan re-scopes them.
   - Owner: `backend-specialist`
   - Est: 3d
   - Deliverables: `schema.snapshot.yaml`, core collection defs (Product, Variant, Service, Booking, Cart, CartItem, Order, OrderSplit, PaymentAttempt, RefundRequest, DeliveryZone, DigitalEntitlement, InventoryReservation, VendorPayoutAccount, TelegramLink, NotificationDelivery, TaxRegistration) with policy row-filter/field-permission rules per `data-model.md`. Encodes the customer-scoped cross-tenant read model (FR-031).

2. [BE] U-T2 — Payment-provider abstraction core  
   - **`status: do-not-implement` (client commerce)** — Fiat = Paddle/Medusa; crypto = undrepay. Do not build parallel Directus payment SoT for shops.
   - Owner: `backend-specialist`
   - Est: 2d
   - Deliverables: provider interface, common `PaymentAttempt` lifecycle, webhook verifier skeleton, async webhook intake + retry/backoff/circuit-breaker + reconciliation-sweep scaffolding (FR-032).

3. [BE] U-T3 — Stripe Connect integration  
   - **`status: do-not-implement` (client commerce / pre-LLC)** — Multi-vendor Connect is post-Stripe phase on Medusa, not Directus `001`.
   - Owner: `backend-specialist`
   - Est: 4d
   - Depends on: U-T2
   - Deliverables: Connect Express onboarding, webhook handlers, application fee handling.

4. [BE] U-T4 — Crypto adapters (TON, SHKeeper)  
   - **`status: do-not-implement` (client commerce)** — Implement crypto via `undrepay` + Medusa provider, not Directus adapters here.
   - Owner: `backend-specialist`
   - Est: 3d
   - Depends on: U-T2
   - Deliverables: checkout quotes, collect-and-forward settlement through per-instance crypto settlement wallets, vendor payout wallet snapshot on `PaymentAttempt`, platform-fee retention during reconciliation, HMAC signature verification, idempotency replay protection, 200 OK webhook handlers, crypto amount-mismatch state handling (`underpaid`/`overpaid`/`expired`, FR-032).

5. [BE] U-T5 — Tax-provider abstraction & Stripe Tax integration  
   - **`status: do-not-implement` (client commerce)** — Paddle MoR owns tax for pre-LLC shop path.
   - Owner: `backend-specialist`
   - Est: 3d
   - Deliverables: tax-provider interface, Stripe Tax adapter, VIES VAT validation hook.

6. [BE] U-T6 — Cart, multi-vendor checkout, fulfilment, refunds & delivery  
   - **`status: do-not-implement` (client commerce)** — Medusa owns cart/order for client shops.
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
   - **`status: do-not-implement` (client commerce)** — Flagship concierge/booking may be re-specified under `003` + Medusa SKUs; do not build client-shop booking on Directus from this task list.
   - Owner: `backend-specialist`
   - Est: 3d
   - Depends on: U-T1, U-T2
   - Deliverables (FR-019, FR-030): slot selection + `pending` booking creation; provider confirm/reject with atomic slot reservation; overlapping-pending-request rejection with concurrency-safe locking (no double booking); reschedule/cancel with slot release; pre-paid booking → linked `PaymentAttempt` (card authorize-on-request/capture-on-confirm/void-on-reject; crypto charge-on-request/auto-refund-on-reject via FR-020 path); pay-later/free bookings create no payment.

8. [BE] U-T8 — Notifications extension
   - Owner: `backend-specialist`
   - Est: 2d
   - Note: Telegram notification patterns may still apply to flagship/housing; do not couple to Directus client-shop checkout events.
   - Deliverables: Directus events → channel adapter (Telegram primary + email fallback), NotificationDelivery audit log, per-channel retry with bounded exponential backoff and no duplicate-on-success (FR-033).

9. [OPS] U-T9a — Provisioning pipeline & admin seed  
   - **`status: do-not-implement` (Directus shop provision)** — Use `002` Medusa manifest + `undevops/003` instead.
   - Owner: `devops-engineer`
   - Est: 2d
   - Deliverables: operator runbook, templated `undevops` deploy pipeline, and admin account seeding with credentials delivered out-of-band (FR-022).

9b. [OPS] U-T9b — ACME TLS & secret management runbook  
   - **`status: do-not-implement` (Directus shop provision)** — Covered by undevops Medusa deploy path.
   - Owner: `devops-engineer`
   - Est: 3d
   - Depends on: U-T9a
   - Deliverables: dual-mode ACME domain & SSL flow (client-owned custom domain + platform-provided wildcard TLS, FR-023), and per-instance secret handling (FR-035): generation, deploy-time injection via `undevops` secret store/env (Stripe keys + webhook signing secrets, TON/SHKeeper credentials), reference-only wiring into extensions, rotation and revocation procedures.

10. [FE] U-T10 — Frontend: `undrllanding` Next.js storefront  
    - **`status: do-not-implement` (from this tasks file)** — Implement under `undrllanding/specs/001-init-storefront-miniapp` against **Medusa Store API**, not Directus.
    - Owner: `frontend-specialist`
    - Est: 5d
    - Depends on: U-T1, U-T6, U-T7, U-T8
    - Deliverables: Next.js storefront with dynamic `@underundre/undesign` theme loading; catalog, cart, checkout, service-request journeys against the versioned contract (`contracts/`).

11. [FE] U-T11 — Telegram Mini App client integration  
    - **`status: do-not-implement` (Wave-1 shop from this list)** — TG Mini App = undrllanding Wave-2; not Directus shop client.
    - Owner: `frontend-specialist`
    - Est: 2d
    - Depends on: U-T1, U-T7, U-T8
    - Deliverables: TG Mini App view, WebApp auth initData verification, shared contract catalog/cart/checkout/booking surface.

12. [TEST] U-T12 — Contract tests & CI  
    - **`status: do-not-implement` (client commerce suite)** — Contract tests for Medusa/Paddle live under undreseller + undrllanding.
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
- Lane E (Provisioning & Docs): U-T9a → U-T9b → U-T15

## Critical Path

**Historical (do not execute for client shops):** U-T1 → U-T6/U-T7 → U-T12.

**Current ecosystem critical path (Gate 0):** undreseller template-shop → undevops Medusa provision → undrllanding Medusa Wave-1 — see `ECOSYSTEM-REQUIREMENTS.md` §5.

## Agent Dispatch Plan

| Owner | Tag | Assigned Tasks | Total Est | Dispatch |
| ----- | --- | -------------- | --------- | -------- |
| `backend-specialist` | `[BE]` | U-T1…U-T7 client paths | — | **blocked / do-not-implement** |
| `backend-specialist` | `[BE]` | U-T8, U-T13 (flagship seams) | 4d | optional flagship only |
| `frontend-specialist` | `[FE]` | U-T10, U-T11 | — | **do-not-implement here** → undrllanding |
| `devops-engineer` | `[OPS]` | U-T9a, U-T9b | — | **do-not-implement** → undevops 003 |
| `test-engineer` | `[TEST]` | U-T12 | — | **do-not-implement** client suite |
| `security-auditor` | `[SEC]` | U-T14 | 2d | only if flagship schema revived |
| `documentation-writer` | `[DOC]` | U-T15 | 1d | only if residual docs needed |

## Total task count: 16 (most marked do-not-implement for client commerce)

## Total estimate: 44 dev-days **historical**; **do not schedule** as client-shop MVP

## Suggested MVP Scope

- **Do not** deliver Directus Tier-A marketplace from this file. Client shop MVP = Medusa template-shop + Paddle + undevops + undrllanding. This tasks file remains historical + extension-seam reference (U-T13) only.
