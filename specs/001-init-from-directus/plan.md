# Implementation Plan: 001-init-from-directus (undrlla)

**Branch**: `001-init-from-directus` | **Date**: 2026-07-18 | **Spec**: `spec.md` (same folder)
**Input**: Rebase `undrlla` onto Directus 12.x (v12.1.1) and deliver a Tier-A marketplace template with extension points for flagship features.

> **Governance note (nested repo + constitution scope):** The repository-root constitution governs the SpecKit pipeline mechanics used here — especially Principle VI (cross-AI review gate) and Principle VII (review-stage tagging) — even though its domain rules for `clai-helpers` do not directly define Directus marketplace architecture. This feature therefore claims compliance with the pipeline principles and treats Source-of-Truth/Transformer/SemVer rules as N/A to marketplace implementation details. RECOMMENDATION: introduce a feature-appropriate constitution scoped to `repos/undrlla` (tenancy isolation, payment/PCI boundaries, contract versioning, PII retention) before `002+`; it should extend domain rules without replacing the root SpecKit gate mechanics.

## Summary

Deliver an operator-provisionable Directus-based marketplace template that:

- Deploys per-client Directus instances via `undevops` using `schema.snapshot.yaml`.
- Exposes REST/GraphQL contract for products, orders, cart, services, bookings.
- Integrates a payment-provider abstraction (Stripe Connect, TON, SHKeeper) and a tax-provider abstraction (Stripe Tax + adapter).
- Provides server-side `Cart`/`CartItem`, guest checkout, per-vendor `Order` splitting with `checkout_group_id`.
- Ships `undrllanding` Next.js storefront and a Telegram Mini App using the same API contract.

## Technical Context

**Primary Platform**: Directus 12.x (pinned v12.1.1) running in container per-instance.  
**Languages**: Directus extensions in Node.js/TypeScript; storefront in Next.js (React/TypeScript).  
**Primary Dependencies**: Directus 12, PostgreSQL (hosted per instance), Stripe (Connect + webhooks), Stripe Tax, TON/SHKeeper adapters, Telegram Bot API, `@underundre/undesign`.  
**Testing**: Contract (API) tests, e2e tests for storefront + Mini App, integration tests for payments and webhooks.  
**Target Platform**: Containerized deployments via `undevops` (Kubernetes/nomad/docker-compose runner).  
**Constraints**: Per-client isolated Directus instance; operator-run provisioning for v1; vendor onboarding flows via admin invites.

## Phase 0 — Research

- Create `research.md` capturing Directus extension patterns, webhook signature verification approaches for each provider, and options for tax/nexus gating.
- Identify existing `undevops` deploy runbook and ACME flow for per-domain TLS issuance.

## Phase 1 — Design & Implementation Plan (Milestones)

1. Directus schema & extension scaffolding: `schema.snapshot.yaml`, collections (Product, Variant, Service, Booking, Cart, CartItem, Order, OrderSplit, PaymentAttempt, RefundRequest, DeliveryZone, DigitalEntitlement, InventoryReservation, VendorPayoutAccount, TelegramLink, NotificationDelivery, TaxRegistration) with the customer-scoped cross-tenant read model. (3d)
2. Payment-provider abstraction & providers: implement Stripe Connect integration (webhooks, Connect Express flows), SHKeeper/TON adapters (wallet address + webhook), async webhook intake with retry/backoff/circuit-breaker + reconciliation and crypto amount-mismatch states. (5d)
3. Tax-provider abstraction: integrate Stripe Tax for card, create adapter for crypto tax rates, and VIES integration. (3d)
4. Cart, checkout, fulfilment, refunds & delivery: server-side cart, multi-vendor splitting, checkout_group_id/OrderSplit, guest checkout, inventory reservation, physical/digital fulfilment (FR-018), refunds (FR-020), delivery zones (FR-021). (4d)
5. Service booking & service checkout: slot selection, atomic confirm/reject, concurrency-safe overlapping-request rejection, reschedule/cancel, pre-paid booking payment linkage (FR-019/FR-030). (3d)
6. Notifications: Directus-events extension with channel adapter (Telegram primary + email fallback) and retry/backoff. (2d)
7. Provisioning runbook & `undevops` templates: operator-run deploy scripts, domain selection flow, seed admin, per-instance secret handling. (2d)
8. Frontend clients: `undrllanding` Next.js storefront + Telegram Mini App contract integration and guest checkout UI. (7d)
9. Tier-B extension-point scaffolding (FR-016 only): isolated extension package seams; no flagship collections/UI (deferred to `002+`). (2d)
10. Security & RLS/FLS audit: cross-tenant leak + cart-visibility tests, FLS blacklist audit, guest token/handle abuse checks. (2d)
11. Tests & CI: contract tests, webhook simulation, e2e scenarios for checkout/bookings. (3d)
12. Quickstart & docs. (1d)

**Estimate (rough)**: **41 developer-days** across multiple engineers (backend, frontend, payments, testing, security, docs) for the Tier-A `001` scope. This reconciles the prior 23–24d figure: the delta is the booking/fulfilment/refund/delivery work now explicitly tasked, the resilience/security tasks, and the removal of the Tier-B flagship collections/UI (~8d) that were deferred to `002+`.

## Deliverables

- `plan.md`, `research.md`, `data-model.md`, `quickstart.md`, `tasks.md`, and the versioned API contract in `contracts/` (`openapi.yaml`).
- Directus extension packages and `schema.snapshot.yaml` for deterministic deployment.
- Payment & tax adapters with integration test harness.
- `undrllanding` storefront and Telegram Mini App using the same API contract.
- Operator runbook for provisioning and domain management.

## Risks & Mitigations

- Stripe Connect KYC / payout flow complexity — mitigate by using Connect Express and deferring complex payout reconciliation to a later iteration.
- Crypto settlement and fee splitting ambiguity — mitigate by defining explicit adapter behavior (collect-and-forward vs direct wallet) per vendor during onboarding.
- Tax legal edge cases — mitigate by gating collection via configurable nexus and deferring complex jurisdictional logic to a tax-provider adapter.

## Next Steps (for `/speckit.tasks`)

- Generate `tasks.md` enumerating tickets for schema, adapters, provisioning, frontend, tests, and assignment to agents.
