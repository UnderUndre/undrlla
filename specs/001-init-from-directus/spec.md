# Feature Specification: 001-init-from-directus (Undrlla Core Rebase on Directus)

**Feature Branch**: `001-init-from-directus`
**Created**: 2026-07-18
**Status**: Approved / In Specification — **partially SUPERSEDED for client shops (2026-07-27)**

> ## ⛔ LEGACY BANNER — client commerce (2026-07-27, hardened 2026-07-29)
>
> **Do not implement client marketplaces on Directus** from this spec. **Do not run `tasks.md` client-shop paths.**
>
> | Concern | Current source of truth |
> |---------|-------------------------|
> | Client shop engine | **`undreseller`** (Medusa) — `undreseller/UNDERUNRE.md` + `undreseller/specs/001-template-shop/` |
> | Cross-repo rules | `undrlla/specs/ECOSYSTEM.md` Q8–Q11 |
> | Launch readiness | `undrlla/specs/ECOSYSTEM-REQUIREMENTS.md` |
> | Storefront | `undrllanding/specs/001-init-storefront-miniapp` (Medusa Store API + Paddle + SHKeeper) |
> | One-click deploy | `undevops/specs/003-undrlla-one-click-deploy` (Medusa + Postgres + Redis) |
> | Provision payload | `002-provisioning-portal` + `contracts/medusa-provisioning-manifest.schema.json` |
> | Fiat (pre–US LLC) | **Paddle** MoR → **Stripe** after US LLC |
> | Crypto | **SHKeeper** via **`undrepay`** (Blnk ledger / test harness) |
> | SSO / JWT | `005-sso-jwt-contract.md` (Zero-Code: Directus native + `@medusajs/auth`) |
> | Pricing / affiliate | `008-pricing-and-affiliate.md` |
>
> **Still valid here:** flagship polity extension points (FR-016), housing-adjacent intent, multi-vendor *intent*, hub/logistics *deferred* FR.  
> **Historical / do not implement for clients:** Tier-A “clean Directus shop template”, Directus product/cart/order SoT for client shops, Directus snapshot one-click for shops.
>
> **Agent rule:** if a task would create a client shop on Directus, **STOP** and implement Medusa template-shop instead.

**Input (historical + retained scope):** Flagship **undrlla** = Directus (or slim API) for **IdP / polity / housing / orchestration**, not client e-commerce SoT. Client shops = **Medusa (`undreseller`)**. Flagship Undrlla.com remains composite (IdP + housing + Medusa SKUs + deferred hub/VPN/logistics).

---

## Executive Context & Architecture Overview

`undrlla` was planned as a transition from legacy Medusa v2 (`repos/undrllegacy`) to Directus (`repos/undrlla`). **2026-07-27 pivot:** client commerce returns to **Medusa via `undreseller`**; Directus (or slim API) remains candidate for **flagship polity + housing**, not for client shop SoT.

### Architectural Philosophy: Two-Tier Model + Marketplace-of-Marketplaces

Undrlla is not only a **site factory** (create + deploy client marketplaces on custom/sub domains). The flagship can also **discover and display** those marketplaces — a **marketplace of marketplaces** — so users browse client stores from Undrlla.com without needing each domain upfront. Provisioning (`002`) creates the instance; a **directory listing** (deferred Tier-B) makes it visible on the hub when the client opts in.

1. **Tier A — Clean Marketplace Engine (`undrlla-core` / Client Template)**
   - **Target Audience**: External businesses, local communities, and solo entrepreneurs.
   - **Scope**: Clean, lightweight E-Commerce (physical & digital products) and Service Marketplace (consultations, appointments).
   - **Multi-Tenancy & Security**: Each client marketplace is deployed as its **own isolated Directus instance/container** (via `undevops`) for hard cross-client isolation; **within** each instance, Directus policy/permission row filters and field permissions separate vendors, customers, and sub-organizations and enforce role-based column blacklisting/whitelisting. Direct database access is restricted to the Directus service and migration identities.
   - **Dynamic Branding**: UI themes, CSS custom properties, Tailwind presets, and DaisyUI themes are dynamically injected via `@underundre/undesign`.
   - **Deployment**: 1-click deployment using `undevops` by applying Directus schema snapshots (`schema.snapshot.yaml`). Domain: client-owned custom domain **or** platform subdomain (FR-023).

2. **Tier B — Undrlla Flagship Platform Instance (`Undrlla.com`)**
   - **Scope (2026-07-18 / 2026-07-22)**: Deferred to follow-up specs (`002+`); `001-init` delivers only the extension-point architecture that Tier B plugs into.
   - **Target Audience**: Global users of the flagship Undrlla platform operated by the founder; workers (couriers, drivers, warehouse owners, housing pros); businesses that need logistics and professional services.
   - **Scope**: Built directly on top of `undrlla-core` as Instance #1, extending it with custom modular plugins:
     - **Mutual Aid Queue ($300k payout pool)**: Queue-based donation system ($1/mo min) with activity-based accelerators (Pressure Points / Flow) and escrow-based direct item/service fulfillment (no cash payouts).
     - **Housing + CBI donations** (`003-housing-donations`): donation-funded property / citizenship-by-investment packages + concierge.
     - **VPN Provisioning (`unet`)**: Rootless Go daemon leveraging `amneziawg-go` and `gVisor netstack` in user-space for DPI-resistant tunneling and port exposure without administrator privileges.
     - **Marketplace hub (federated catalog + unified cart, Hub MoR)**: Flagship indexes opt-in client catalogs; unified cart; **Undrlla is Merchant of Record** (customer pays flagship → settle to vendors). Create+deploy **and** full catalog + cart.
     - **Logistics work network (simple v1)**: Self-signup workers; price-first match; **hold on accept / capture on complete**; **12%** fee; shared pool. Not full Uber/WMS.
     - **Housing / CBI workforce**: Lawyers, realtors, translators, notary-escorts self-signup (post-`003` multi-vendor) as ordinary `Service` / vendor providers.
     - **P2P Barter Board**: Item/service swap listings charging a minimal listing fee (TON / Stars / Fiat) and linking via `undrepost` messaging.
     - **AI Component Builder**: Smart PC assembly recommendation engine.

### Logistics & workforce product intent (simple first)

| Role | What they offer | Who hires them | Simple v1 |
|------|-----------------|----------------|-----------|
| **Courier** | Last-mile delivery jobs | Store / business with orders | Post job (pickup/dropoff, price or bid) → worker accepts → status updates |
| **Taxi / driver** | Passenger trips | Riders / businesses needing rides | Same match pattern as courier with origin/destination geo |
| **Warehouse owner** | Storage slots / short-term storage | Stores needing inventory hold | Listing: location, capacity, price/day or price/m³ → book slot |
| **Housing pro** | Legal, realtor, translation, escort | Housing/CBI applicants | Bookable `Service` rows (reuse FR-019); multi-vendor post-`003` MVP |

**Out of simple v1:** full fleet routing, live GPS tracking fleets, multi-warehouse WMS, dynamic surge pricing, automated dispatch algorithms. Reviews may ship as a thin follow-on after price-sorted matching.

---

## Clarifications

### Session 2026-07-18

- Q: What is the primary multi-tenancy isolation model — one shared Directus instance with `tenant_id` row filters, or an isolated instance/container per client? → A: Hybrid — each client marketplace is its own isolated Directus instance/container (deployed via `undevops`) for hard cross-client isolation; Directus policy/permission row filters and field permissions operate within each instance to separate vendors, customers, and sub-organizations.
- Q: Are the Tier-B flagship modules (Mutual Aid, VPN, Taxi, Barter, AI builder) in scope for `001-init`? → A: No — deferred to follow-up specs (`002+`). `001-init` delivers the Tier-A core plus the extension-point architecture they plug into; FR-007–FR-012, User Story 3, and SC-004/SC-005 are out of scope for this feature.
- Q: How do users authenticate into a Tier-A marketplace? → A: Directus native authentication (email/password with refresh tokens in `httpOnly` cookies) plus Admin/Vendor/Customer roles feeding `$CURRENT_USER` for Directus row filters; OAuth/SSO providers are optional per-instance config. Telegram-based auth is deferred to the flagship.
- Q: Which payment gateway is the v1 primary for the Tier-A marketplace? → A: All three — Stripe Connect, SHKeeper (crypto), and TON — are first-class in `001`, each implemented behind a common payment-provider abstraction (uniform checkout / webhook / refund interface).
- Q: Which Directus major version does `001-init` target? → A: Directus 12.x, pinned to the current stable v12.1.1; `schema.snapshot.yaml` and extensions target the 12 major, and version bumps are deliberate tested migrations (not floating).
- Q: What does `tenant_id` represent under the two-tier isolation model? → A: A sub-organization or storefront within one client-owned Directus instance; each client marketplace is separately isolated in its own instance/container.
- Q: Which layer enforces tenant access controls? → A: Directus policy/permission row filters and field permissions enforce REST/GraphQL access; direct database access is restricted to the Directus service and migrations.
- Q: How is payment state recorded across the three v1 providers? → A: A shared `PaymentAttempt` collection stores state, an idempotency key, provider identifiers, and webhook event identifiers; a server-side extension verifies webhook signatures and advances the attempt through a common state machine.
- Q: Which storefront clients are delivered in `001-init`? → A: `undrllanding` Next.js storefront + Telegram Mini App shell share the API contract. **Wave-1 auth:** Directus email/password (and optional OAuth). **Telegram Mini App login (`initData` → JWT) is Wave-2** — implemented together with FR-017 flagship extension; until then Mini App may use linked web session or email login. undrllanding MUST NOT require TG auth for Wave-1 shop.
- Q: How are physical and digital products fulfilled in v1? → A: Physical products use inventory, manual fulfillment, and an optional tracking number; digital products grant a protected download link only after confirmed payment.
- Q: What is the v1 service-booking flow? → A: A customer selects an available slot and creates a `pending` booking; the provider manually confirms or rejects it, and either party can cancel or reschedule it.
- Q: How are simultaneous booking requests for the same slot resolved? → A: Multiple requests may remain `pending`; confirming one booking atomically reserves the slot and rejects all overlapping pending requests.
- Q: How is physical inventory protected from overselling during checkout? → A: Checkout atomically creates an inventory reservation; confirmed payment consumes it, while cancellation, payment failure, or TTL expiry releases it.
- Q: How are refunds authorized in v1? → A: A customer creates a refund request; a vendor or administrator approves a full or partial amount, then the common payment adapter calls the provider.
- Q: Which physical-delivery model does v1 support? → A: A vendor configures delivery zones with fixed or free rates and local pickup; a delivery address is required when the customer selects delivery.
- Q: How is a client marketplace provisioned in `001`? → A: Operator-run — the operator triggers `undevops` deploy (CLI/pipeline) using a documented runbook; deploy seeds the first Admin (credentials delivered out-of-band). A self-service order portal and per-instance billing are deferred to a dedicated `002-provisioning-portal` spec.
- Q: Where does the client marketplace domain come from? → A: The client MUST declare at provisioning time whether they bring their own domain (client purchases/owns it and points DNS at the platform; platform issues per-domain TLS via ACME) or the platform provides a subdomain under its apex (operator-managed DNS + wildcard TLS). Both MUST be supported.
- Q: How are multi-vendor carts structured into orders and payments? → A: An `Order` is always single-vendor; a cart may span vendors and is split at checkout into one `Order` + one `PaymentAttempt` per vendor (linked by a shared `checkout_group_id`), each paid, fulfilled, refunded, and paid out independently. The platform fee is applied per vendor order.
- Q: How are vendors onboarded and paid out, and how is the platform fee taken? → A: **Tier-A client marketplaces (v1):** Admin-invited vendors only (no public vendor self-signup). **Flagship Undrlla.com workforce** (couriers, taxi, warehouse, housing pros — FR-038/039): **self-signup** + payout KYC (Connect/crypto). Each completes payout setup per provider — Stripe Connect Express and TON/SHKeeper wallet. Platform fee % per vendor/job order. A payment method is offered only if payout setup is complete.
- Q: What is the primary notifications channel? → A: A Telegram bot is the **primary** channel for all transactional notifications, driven by a Directus-events notification extension over a channel-adapter interface. Recipients link their Telegram chat to their account (Mini App via `initData`, web storefront via a bot deep-link). **Email is the fallback** for recipients without a linked Telegram, so no event is silently dropped. Linking a chat for delivery is distinct from Telegram-based authentication (still deferred, FR-017).
- Q: How is the cart persisted and is guest checkout allowed? → A: Server-side `Cart`/`CartItem` keyed by the authenticated user or an anonymous cart token (cookie), syncing across the web storefront and Mini App. Guest checkout is allowed with a required contact handle (Telegram or email) for notifications and optional post-purchase account creation; an anonymous cart merges into the user's cart on login.
- Q: How is currency handled across fiat and crypto providers? → A: One canonical settlement currency per instance (set at provisioning); `Product.price` and order totals use it. Cards settle in it directly; crypto (TON/SHKeeper) converts at checkout via a rate quote with a short TTL that locks the crypto amount (re-quote on expiry). The quoted rate, crypto amount, settlement currency, and quote expiry are recorded on `PaymentAttempt`.
- Q: How is tax computed for physical + digital sales across providers? → A: An external tax engine behind a tax-provider abstraction — Stripe Tax for card orders; a pluggable rates adapter + official VIES VAT-number validation (EU B2B reverse-charge → 0%) for crypto/non-Stripe orders. Collection is gated by configurable nexus/OSS thresholds; computed tax is recorded on the `Order`. Merchant-of-Record engines (Paddle / Lemon Squeezy) are incompatible with the multi-vendor Stripe Connect payout model and are out of scope for the marketplace configuration.
- Q: When and how is a service booking paid? → A: Per-`Service` policy — pre-paid or pay-later/free. Pre-paid links `Booking`→`PaymentAttempt` via the payment abstraction: cards authorize on request and capture on confirm (void on reject); crypto charges on request and auto-refunds on reject (FR-020 path). Pay-later/free bookings carry no payment. Confirmation atomically captures payment and reserves the slot.

### Session 2026-07-21

- Q: How are concierge / personal-assistant offerings (e.g. airport pickup & escort to property signing, proxy purchases via a local bank card, rental-handover help, legal/banking setup) modeled in a Tier-A instance? → A: As ordinary `Service` records with `provider_id` set to the vendor offering them and a per-service `payment_policy` (`prepaid` or `pay_later_free`); each offering is its own `Service` row, booked through the standard `Booking` flow (FR-019/FR-030). No dedicated collection is required in `001`. Geographic availability of such services is governed per-instance by the `ServiceArea` collection introduced in spec `003-housing-donations` (Tier-B); until `003` lands, a Tier-A instance simply attaches concierge services to a tenant without geo-scoping.
- Q: Where does the purchase of housing funded by donations (escrow payout to a notary / verified escrow agent, never raw cash to the recipient) get specified? → A: Not in `001`. It is a Tier-B flagship extension modeled as first-class `HousingProgram` / `HousingApplication` / `HousingEscrow` / `HousingPoolAllocation` collections, with admin-controlled per-country availability via the universal `ServiceArea` collection, specified in `003-housing-donations`. It builds directly on the escrow pattern of FR-011 (no unverified cash transfers) and on the `PaymentAttempt` / `Order` / `OrderSplit` machinery of `001`. The Tier-A `001-init` ships none of these collections by default (FR-016 still holds).

### Session 2026-07-22

- Q: Does Undrlla only create/deploy client sites, or also show them? → A: **Both.** Full catalog search + **unified cart**; commerce model is **Hub Merchant of Record**: customer pays flagship Undrlla, flagship settles to client vendors (Connect/crypto). Inventory revalidated at cart/checkout. Build in `004-marketplace-hub`.
- Q: Logistics workers? → A: Self-signup; shared flagship pool; jobs from hub or client shop; Connect/crypto; **12%** fee; **hold on accept → capture on complete** (void on cancel; dispute blocks capture). Build `005`.
- Q: Housing donor model? → A: **Shared pot + queue** (subscription into pool); hero outcome housing+CBI/passport; **no earmark** in MVP. Donation fee **5%** (`003`).
- Q: Housing/CBI helpers? → A: Self-signup post-`003` multi-vendor; ordinary marketplace fee.
- Q: What is not in simple logistics v1? → A: Fleet routing, live tracking, surge, insurance product, full WMS.

---

## User Scenarios & Acceptance Criteria *(mandatory)*

### User Story 1 — Deploying a Clean Client Marketplace (Priority: P1)

As a business owner or client, I want to deploy a clean, branded e-commerce and services marketplace for my business using `undevops`, so that I can sell products and services without unwanted platform bloat or complex manual server setup.

**Why this priority**: Core value proposition of the productized template. If client deployment fails or includes unwanted platform extensions, the commercial model fails.

**Independent Test**: Can be tested by executing a single `undevops` deploy command targeting a fresh Directus container, applying `schema.snapshot.yaml`, picking a brand theme from `@underundre/undesign`, and verifying that a clean storefront is accessible with product/checkout functionality.

**Acceptance Scenarios**:

1. **Given** a fresh Docker environment managed by `undevops`, **When** the `undrlla-core` deployment workflow triggers, **Then** Directus initializes, applies `schema.snapshot.yaml`, and starts serving REST and GraphQL APIs for products, orders, and services.
2. **Given** a deployed client instance, **When** inspecting the schema and admin UI, **Then** no Undrlla-specific flagship modules (Mutual Aid, VPN Nodes, Taxi Board) are present or enabled by default.
3. **Given** a client storefront configured with theme tokens from `@underundre/undesign`, **When** the storefront renders, **Then** CSS variables and Tailwind classes match the chosen client brand color palette instantly without code recompilation.
4. **Given** a valid provider webhook is delivered more than once, **When** the payment extension processes it, **Then** it verifies the signature and records the event idempotently, resulting in at most one state transition for the matching `PaymentAttempt` and `Order`.
5. **Given** a deployed client marketplace, **When** a customer uses either the `undrllanding` Next.js storefront or the Telegram Mini App, **Then** they can browse the catalog, manage a cart, complete checkout, and request services through the same versioned Directus API contract.
6. **Given** an order with confirmed payment, **When** it contains physical products, **Then** a vendor can fulfil it manually and record an optional tracking number; **When** it contains digital products, **Then** the customer receives a protected download link.
7. **Given** a customer selects an available service slot, **When** they submit a booking request, **Then** the booking starts in `pending`; the provider can confirm or reject it, and either party can cancel or reschedule it through the shared API contract.
8. **Given** two or more overlapping `pending` booking requests, **When** a provider confirms one, **Then** the service slot is atomically reserved for that booking and every overlapping pending request is rejected.
9. **Given** a customer starts checkout for the final unit of a physical product, **When** the inventory reservation is created, **Then** another checkout cannot reserve that unit; **When** payment fails, is cancelled, or the reservation expires, **Then** the unit becomes available again.
10. **Given** a customer requests a full or partial refund for a paid order, **When** the vendor or administrator approves it, **Then** the common payment adapter submits the approved amount to the selected provider and records the resulting state without exceeding the captured amount.
11. **Given** a customer checks out with physical products, **When** they select delivery, **Then** the system requires a delivery address and applies the configured rate for the matching delivery zone; **When** they select local pickup, **Then** no delivery address or shipping rate is required.
12. **Given** a client declares a domain source at provisioning, **When** they bring their own domain, **Then** deploy expects client-configured DNS pointing at the platform and issues per-domain TLS via ACME; **When** the platform provides a subdomain, **Then** deploy assigns an operator-managed subdomain served under the platform's wildcard TLS.
13. **Given** a cart containing items from two different vendors, **When** the customer checks out, **Then** the system creates one `Order` and one `PaymentAttempt` per vendor sharing a single `checkout_group_id`, and each order can be paid, fulfilled, and refunded independently.
14. **Given** a vendor has not completed Stripe Connect Express onboarding, **When** a customer reaches checkout for that vendor's order, **Then** card payment via Stripe is not offered for that order; **When** onboarding is complete, **Then** the platform fee is applied and the vendor is paid out via their connected account.
15. **Given** an order's payment is confirmed, **When** the notification fires, **Then** a recipient with a linked Telegram receives a bot message; **When** the recipient has no linked Telegram, **Then** the same notification is delivered by email fallback and no event is silently dropped.
16. **Given** a guest with items in an anonymous cart, **When** they check out and provide a contact handle, **Then** the order is created without requiring an account and notifications are delivered to that handle; **When** the guest later logs in or registers, **Then** their anonymous cart merges into their account cart.
17. **Given** an instance canonical currency and a customer paying via a crypto provider, **When** checkout begins, **Then** the system quotes the crypto amount from the canonical price with a TTL and records the rate on the `PaymentAttempt`; **When** the quote TTL expires before payment, **Then** the system re-quotes rather than accepting a stale rate.
18. **Given** a checkout in a jurisdiction where the instance is registered to collect tax, **When** the customer pays by card, **Then** Stripe Tax computes the tax and it is recorded on the order; **When** they pay by crypto, **Then** the tax adapter computes it from the rates provider; **When** an EU business supplies a VIES-valid VAT number, **Then** reverse-charge applies and tax is 0%.
19. **Given** a pre-paid service, **When** a customer requests a booking, **Then** a linked `PaymentAttempt` is created (card authorized, or crypto charged); **When** the provider confirms, **Then** the payment is captured and the slot is atomically reserved; **When** the provider rejects, **Then** the card authorization is voided or the crypto charge auto-refunded; **Given** a pay-later/free service, **When** a booking is requested, **Then** no payment is created.

---

### User Story 2 — Granular Multi-Tenancy & Data Security via Directus Policies (Priority: P2)

As a marketplace administrator or vendor, I want Directus policy/permission row filters to prevent data leaking between tenants/vendors and field permissions to hide sensitive internal fields, so that user privacy and enterprise data isolation are guaranteed.

**Why this priority**: Essential for multi-tenant safety and compliance. Prevents tenant data spills and role permission exploits.

**Independent Test**: Create two tenant accounts (Tenant A and Tenant B) and three roles (Admin, Vendor, Customer). Query the `/items/products` and `/items/orders` API endpoints under each context to verify row filtering and field stripping.

**Acceptance Scenarios**:

1. **Given** Vendor A (belonging to Tenant A) requesting `/items/orders`, **When** the Directus engine evaluates its policy/permission filters, **Then** only orders where `tenant_id == Vendor A.tenant_id` are returned.
2. **Given** a Customer role requesting product details, **When** the response is serialized, **Then** fields marked in the FLS blacklist (e.g. `wholesale_margin`, `internal_supplier_notes`) are completely omitted from the JSON payload.
3. **Given** an attempt by Vendor A to modify an item belonging to Vendor B, **When** the PATCH request hits the API, **Then** Directus rejects the request with a 403 Forbidden error before the mutation reaches the database.

---

### User Story 3 — Flagship Instance Extensions: Mutual Aid Queue & Rootless VPN (`Undrlla.com`) (Priority: P3)

> **Scope note (2026-07-18):** This story targets Tier-B flagship modules **deferred to follow-up specs (`002+`)** and is retained for context; `001-init` only delivers the extension-point architecture that will host it.

As a user on the flagship Undrlla.com platform, I want to join the $300k Mutual Aid Queue, boost my queue position by participating in ecosystem activities (e.g. hosting a `unet` VPN node or offering services), and purchase VPN subscriptions.

**Why this priority**: Defines the unique value proposition and monetization engine of the founder's flagship platform.

**Independent Test**: Test queue registration with a $1/mo subscription trigger, verify score calculation based on activity logs in `vpn_nodes`, and test user-space tunnel establishment via `unet`.

**Acceptance Scenarios**:

1. **Given** an active user who donates $1/month, **When** they request entry into the Mutual Aid Queue with a verified need (e.g. passport or housing request), **Then** they are appended to the queue with an initial score.
2. **Given** a user running a `unet` node on their PC/router, **When** their node reports uptime via `amneziawg-go`, **Then** the platform awards "Pressure Points / Flow", advancing their priority in the Mutual Aid Queue.
3. **Given** a queue recipient whose goal ($300k pool) is reached, **When** payout processing occurs, **Then** the platform disburses funds directly to escrow/service providers (e.g. housing escrow, legal passport agency) rather than sending unverified raw cash.

---

## Edge Cases

- **What happens when a client tenant attempts to enable flagship modules?**
  - Flagship extensions (Mutual Aid, `unet` management, Taxi board) are isolated in separate Directus extension packages and schema migrations, preventing client instances from accidentally executing unconfigured code.
- **How does the system handle high-volume dynamic policy-filtered queries in Directus?**
  - Database indexes are automatically created on `tenant_id`, `owner_id`, and status fields. GraphQL query depth limiters and role-aware Redis caching prevent database exhaustion (`EXPLAIN ANALYZE` benchmarks enforced).
- **How does the system handle duplicate or reordered payment webhooks?**
  - The payment extension verifies each provider signature, deduplicates events by provider event identifier, and advances the matching `PaymentAttempt` only through valid state transitions.
- **What happens when webhook processing fails or a provider stops delivering?**
  - Webhooks are persisted on receipt and processed asynchronously; transient failures retry with bounded exponential backoff, a per-provider circuit breaker sheds to a dead-letter/reconciliation queue after a failure threshold, and a periodic reconciliation sweep polls provider state for attempts stuck non-terminal past a timeout (FR-032).
- **What happens when a crypto payment arrives under, over, or after the quoted amount/TTL?**
  - The `PaymentAttempt` records an explicit mismatch state: `underpaid` awaits top-up until `quote_expires_at`, `overpaid` flags a partial refund via the FR-020 path, and `expired` forces a re-quote; an order is never marked paid unless the received amount satisfies the quote within tolerance (FR-032).
- **What happens when a notification channel delivery fails?**
  - Primary Telegram delivery retries with backoff before email fallback, the fallback retries with backoff, and each attempt is recorded on `NotificationDelivery` so no catalogued event is silently dropped and a succeeded delivery is not duplicated (FR-033).
- **What happens if a digital download link is shared or expires?**
  - Download access is granted only through the purchaser's authenticated entitlement; a protected link is time-limited and can be revoked by an administrator or vendor.
- **What happens when a booking is rejected, cancelled, or rescheduled?**
  - The booking records its terminal or replacement state; the previously requested slot is released when applicable, and the customer and provider can see the updated status through both storefront clients.
- **What happens when a provider confirms overlapping booking requests concurrently?**
  - Confirmation is atomic: one booking reserves the slot and all overlapping `pending` requests transition to `rejected`, preventing double booking.
- **What happens when checkout ends without confirmed payment?**
  - The matching inventory reservation is released on cancellation, payment failure, or expiry, returning the quantity to available stock; only confirmed payment consumes the reservation into a completed order.
- **What happens when a multi-vendor checkout has mixed outcomes?**
  - Checkout fan-out is independent per vendor order: a successfully paid vendor order remains committed even if a sibling vendor order in the same `checkout_group_id` fails, expires, or is refunded. Failed or expired vendor orders release only their own inventory reservations and surface as `partially_failed` / mixed status in the shared checkout-group view; refunds apply per vendor order without rolling back unrelated paid orders.
- **What happens when a refund request is rejected or a provider refuses the refund?**
  - The refund request records its decision and provider outcome without changing the captured payment amount; only a provider-confirmed refund updates the corresponding `PaymentAttempt` and `Order` refund state.
- **What happens when no delivery zone matches the customer address?**
  - Delivery is unavailable for that address; the customer may choose another address, a configured pickup option, or abandon checkout. The system MUST NOT apply an arbitrary fallback rate.
- **What happens if a user running `unet` lacks Administrator / root privileges on Windows/macOS?**
  - `unet` operates in user-space using `amneziawg-go` with `gVisor netstack`. It binds local ports and exposes a local SOCKS5/HTTP proxy on `localhost:1080` without invoking OS network driver setup or elevated privileges.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST run on Directus **12.x** (Node.js/TypeScript) as the primary headless API and metadata engine, pinned to the current stable **v12.1.1** at implementation time. `schema.snapshot.yaml` and all extensions MUST target the Directus 12 major; version bumps MUST be deliberate, tested migrations and MUST NOT float to newer majors automatically.
- **FR-002**: System MUST store client template schemas in `schema.snapshot.yaml` for deterministic 1-click deployment via `undevops`.
- **FR-003**: Within each client-owned Directus instance, System MUST enforce policy/permission row filters on all collections using `$CURRENT_USER` context (`tenant_id`, `owner_id`), where `tenant_id` identifies a sub-organization or storefront. Each client marketplace is isolated separately at the instance/container level as required by FR-015. REST and GraphQL clients MUST access tenant data only through Directus; direct database access is restricted to the Directus service identity and migration tooling. Customer-scoped collections that intentionally span vendors (`Cart`/`CartItem`, and the customer's read view of their own `Order`s) are the explicit exception governed by FR-031 and MUST be filtered by customer identity rather than by `tenant_id`.
- **FR-004**: System MUST enforce Directus field permissions (blacklists and whitelists) on policies to strip sensitive columns from API outputs.
- **FR-005**: System MUST integrate `@underundre/undesign` for dynamic theme selection (Tailwind presets, DaisyUI themes, CSS variables).
- **FR-006**: System MUST provide REST and GraphQL endpoints for products, orders, cart, and services in the client template layer.
- **SCOPE — Deferred (2026-07-18 / 2026-07-22)**: FR-007 through FR-012 and FR-037 through FR-039 below specify Tier-B flagship modules that are **out of scope for `001-init`** and deferred to follow-up specs (`002+` / dedicated feature specs). They remain listed for traceability only and MUST NOT be implemented in this feature.
- **FR-007**: System MUST support a P2P Barter listing module where users pay a flat publication fee (TON / Stars / Fiat) to list swap offers.
- **FR-008**: System MUST provide a **simple logistics match board** for **taxi drivers and couriers**: a requester posts a job (origin/destination or pickup/dropoff, offered price or open bid); eligible workers see jobs in their service area; a worker accepts; parties exchange contact or in-app status; platform MAY take a micro-fee per completed match. Simple v1 MUST support selection by **price**; **reviews/ratings** MAY follow in a later iteration of the same module. Full fleet routing and live tracking are out of simple v1.
- **FR-009**: System MUST manage the $300k Mutual Aid Queue for the flagship instance, enforcing a $1/mo minimum donor eligibility rule.
- **FR-010**: System MUST calculate queue acceleration ("Pressure Points / Flow") based on verified user contributions (VPN uptime, service fulfillment, and optionally completed logistics / housing-service jobs).
- **FR-011**: System MUST process Mutual Aid payouts exclusively through verified third-party escrow or direct service providers (no direct unverified cash transfers).
- **FR-012**: System MUST integrate `unet` VPN provisioning using `amneziawg-go` and `gVisor netstack` in user-space (rootless execution).
- **FR-037** *(Deferred — federated marketplace hub + unified cart; Hub MoR; see FR-042 phasing)*: Flagship MUST support a **Marketplace Hub** for opt-in client marketplaces so Undrlla **indexes, searches, and sells** their products/services under a **Merchant-of-Record** model when 037b is enabled.
  - **(a) Opt-in**: `list_on_hub` / `share_catalog_to_hub` (`002`).
  - **(b) Catalog sync**: product/service summaries into flagship index (source instance/item ids, price, currency, availability hint, `indexed_at`).
  - **(c) Search/browse** on Undrlla.com.
  - **(d) Unified cart**: multi-instance lines; guest/auth rules at flagship scope.
  - **(e) Hub MoR checkout**: the **customer pays the flagship Undrlla entity** (single or multi-`PaymentAttempt` as needed for mixed rails). Flagship is seller-of-record for tax/receipts/chargebacks on hub sales. After capture, platform **settles net proceeds to source vendors** via Stripe Connect application-fee / crypto split (FR-025 pattern), recording per-line `source_instance_id` + vendor payout. Platform fee applies per line/vendor. Mixed line failures MUST NOT silently oversell; paid lines settle; failed lines do not charge (or refund if already authorized).
  - **(f) Inventory**: revalidate at add-to-cart and checkout against source instance; fail closed.
  - **(g) Currency**: display strategy documented in hub spec; settlement may convert with quoted TTL (FR-028).
  - **(h) Tax**: computed under **flagship MoR** nexus/rules for hub sales (not each client’s nexus alone); vendor settlement is typically net of tax handling as defined in hub tax policy.
  - **(i) Fulfilment**: source instance/vendor still fulfils physical/digital goods; hub order creates or syncs a fulfilment instruction to the source (write-through or hub-owned order with vendor task). Vendor never becomes MoR to the end customer for hub-originated sales.
  - Cards-only or pure handoff (client MoR) is **insufficient**. Out of scope for `001` implementation.
- **FR-038** *(Deferred — logistics + warehouse workforce)*: Self-signup workers (`courier` | `taxi_driver` | `warehouse_owner`) with Connect/crypto payouts. Jobs: post → **accept** → **complete**. Selection by price (reviews later). **Payment lifecycle (locked):** card **authorize/hold on accept** (or at post if the offer is prepaid); **capture on complete** after requester confirmation (or dual-confirm); reject/cancel/no-show **voids** the hold; open dispute **blocks capture** until resolved. Crypto: charge to platform escrow wallet on accept equivalent, release on complete / refund on cancel (document rail limits). Platform fee **12%** of job amount (configurable; optional minimum) taken at capture. Jobs from flagship UI or client “deliver this order”; **shared flagship pool**. Full WMS/fleet out of simple v1.
- **FR-039** *(Deferred — housing/CBI professional workforce)*: System MUST allow professionals who support housing and citizenship-by-investment flows (e.g. lawyers, realtors, translators, document runners, notary-escorts) to **self-signup** as vendors and publish bookable `Service` offerings with Stripe Connect / crypto payouts. Multi-vendor selection by price (and later reviews) reuses FR-019/FR-030; platform fee is the ordinary marketplace fee (FR-025), not a separate logistics fee. `003` MVP remains founder-only and expands later without schema fork.
- **FR-013**: System MUST integrate Stripe Connect, SHKeeper (crypto), and TON payments as first-class payment gateways in v1, each implemented behind a **common payment-provider abstraction** exposing a uniform checkout, webhook, and refund interface so the marketplace core stays provider-agnostic and every gateway is independently testable. A shared `PaymentAttempt` collection MUST record the common payment state, idempotency key, provider identifiers, and webhook event identifier; a server-side extension MUST verify provider webhook signatures, deduplicate events, and advance attempts only through valid common state transitions.
- **FR-014**: System MUST deliver both a full `undrllanding` Next.js storefront and a Telegram Mini App. Each client MUST implement catalog browsing, cart management, checkout, and service requests through the same versioned OpenAPI / GraphQL contract exported by the platform.
- **FR-015**: System MUST provision each client marketplace as an isolated Directus instance/container via `undevops` (container-level cross-client isolation). Within a single instance, Directus policy/permission row filters and field permissions MUST separate vendors, customers, and sub-organizations; cross-client isolation MUST NOT depend on `tenant_id` filtering alone.
- **FR-016**: System MUST provide the extension-point architecture (Directus extension packages + isolated schema migrations) that the deferred Tier-B flagship modules plug into, such that a clean Tier-A client instance contains none of those modules by default (satisfying User Story 1, Acceptance Scenario 2).
- **FR-017**: System MUST authenticate users via Directus native authentication (email/password with refresh tokens stored in `httpOnly` cookies) and MUST define Admin, Vendor, and Customer roles that drive policy/permission row filters and field permissions through `$CURRENT_USER`. OAuth/SSO providers MAY be enabled per-instance via configuration. **Telegram Mini App authentication** (`initData` HMAC verify → issue JWT/session) is a **flagship Wave-2** extension: MUST be specified and tested before undrllanding enables TG-only login; **out of Wave-1 shop MVP**. Until Wave-2, Mini App uses the same email/password or magic-link session as web.
- **FR-040** *(Role matrix)*: **Client Tier-A instances:** vendor accounts are Admin-invited only (FR-025). **Flagship instance:** roles `courier`, `taxi_driver`, `warehouse_owner`, `housing_pro` MAY self-signup subject to KYC/payout gates (FR-038/039). Flagship may still invite classic product vendors. Specs/implementations MUST NOT apply self-signup to Tier-A client shops by default.
- **FR-041** *(Delivery modes)*: At checkout for physical goods, the customer selects **exactly one** fulfilment path: (a) vendor delivery zone / pickup (FR-021), or (b) platform logistics job (FR-038) when enabled on flagship/opt-in client. System MUST NOT charge both for the same order line.
- **FR-042** *(Hub phasing)*: FR-037 is split for delivery: **(037a)** catalog sync + search (no MoR charge); **(037b)** Hub MoR unified checkout — requires compliance checklist (tax registrations, Stripe MoR setup). 037a MAY ship without 037b.
- **FR-043** *(unet billing — flagship)*: Flagship MUST support paid `unet` subscriptions: create/revoke peer configs, device limits, billing via existing payment abstraction; write local/remote config material for the `unet` client. Engine binary remains the `unet` repo; control plane is undrlla.
- **FR-018**: System MUST support physical and digital product fulfilment in v1. Physical products MUST track inventory, allow manual vendor fulfilment, and support an optional shipment tracking number. Checkout MUST atomically create a time-limited inventory reservation; confirmed payment MUST consume it, while cancellation, payment failure, or reservation expiry MUST release it. Digital products MUST create a purchaser-owned entitlement only after confirmed payment and expose content through a protected, time-limited download link that an administrator or vendor can revoke.
- **FR-019**: System MUST support service booking in v1. A customer MUST be able to select an available slot and create a `pending` booking; the service provider MUST be able to confirm or reject it. Multiple overlapping requests MAY be `pending`, but confirming one booking MUST atomically reserve its slot and reject all overlapping pending requests. Either party MUST be able to cancel or reschedule a booking, with its state and the availability of the previously requested slot updated through the common API contract.
- **FR-020**: System MUST support customer-initiated full and partial refund requests for paid orders, as well as system-initiated auto-refunds (`initiator=system`) for booking rejections or overpayment reconciliation. A vendor or administrator MUST approve or reject customer requests before the payment adapter calls the provider; system-initiated refunds execute automatically. The approved amount MUST NOT exceed the captured amount; only provider-confirmed refunds MAY update the `PaymentAttempt` and `Order` refund state.
- **FR-021**: System MUST support physical-product delivery through vendor-configured delivery zones with fixed or free rates, plus local pickup. Checkout MUST require a delivery address and apply the matching zone rate when delivery is selected; it MUST require neither an address nor a shipping rate for local pickup. If no zone matches, delivery MUST be unavailable rather than using a fallback rate. Interaction with platform logistics is governed by FR-041.
- **FR-022**: System MUST provision each client marketplace via an operator-run `undevops` deploy (CLI/pipeline) documented by a repeatable runbook, and MUST seed the initial Admin account during deploy with credentials delivered out-of-band. A self-service order/onboarding portal and per-instance (marketplace subscription) billing are OUT OF SCOPE for `001-init` and deferred to a dedicated `002-provisioning-portal` spec. Provisioning MUST inject per-instance payment and webhook secrets per the secret-handling model of FR-035.
- **FR-023**: At provisioning time the client MUST declare the domain source. The system MUST support both (a) a **client-owned custom domain** — the client registers/purchases it and points DNS at the platform, and the platform issues per-domain TLS certificates via ACME — and (b) a **platform-provided subdomain** under the platform apex, served with operator-managed DNS and wildcard TLS. The instance's public base URL (used for checkout, `httpOnly` auth cookies, and the exported API contract) MUST match the client's chosen domain.
- **FR-024**: System MUST treat each `Order` as single-vendor (exactly one `tenant_id`). A customer cart MAY contain items from multiple vendors; at checkout the system MUST split it into one `Order` and one `PaymentAttempt` per vendor, linked by a shared `checkout_group_id`. Each resulting order MUST be paid, fulfilled, refunded, and paid out independently through the common payment-provider abstraction, and the platform fee MUST be applied per vendor order. A mixed checkout outcome MUST NOT roll back already-paid sibling vendor orders: each failed/expired/refunded vendor order releases or mutates only its own reservations/payment state, while the checkout-group view exposes aggregate mixed statuses (`partially_paid`, `partially_failed`, `partially_refunded`) to the customer.
- **FR-025**: System MUST onboard **Tier-A product vendors** by Admin invitation (no public vendor self-signup on client instances in v1). Flagship workforce self-signup is FR-040/038/039. Before a vendor/worker can be paid out, they MUST complete payout setup for each active provider: **Stripe Connect Express** hosted onboarding (Stripe performs KYC and holds the connected payout account) and a **payout wallet address** for TON and SHKeeper. The platform fee MUST be a configured percentage applied per vendor order and recorded in `Order.platform_fee_amount` — taken via the Stripe Connect application fee for card payments. For TON and SHKeeper v1, the platform supports both `crypto_collect_and_forward` and `direct_to_vendor` settlement modes (deployments operating without a custodial MSB/MiCA license MUST select `direct_to_vendor`). Every `PaymentAttempt` MUST return structured `payment_instructions` (deposit address, exact crypto amount, quote expiration, memo) so clients can complete checkouts. The payout wallet address MUST be snapshotted on the `PaymentAttempt` at checkout; a vendor wallet change applies only to future payment attempts and MUST NOT alter existing orders/refunds. A provider MUST NOT be offered at checkout for a vendor order unless that vendor has completed payout setup for it. The provider credentials and webhook signing secrets backing this integration MUST be stored and injected per the secret-handling model of FR-035.
- **FR-026**: System MUST deliver transactional notifications for a defined event catalog — payment confirmation, order fulfilment and tracking, digital-download availability, booking requested/confirmed/rejected/rescheduled/cancelled, and refund decision — routed by recipient role (customer, vendor, provider, admin) via a Directus-events notification extension over a channel-adapter interface. A **Telegram bot MUST be the primary channel**; each recipient links their Telegram chat identifier to their account (Mini App via `initData`, web storefront via a bot deep-link). **Email MUST serve as the fallback** channel for recipients without a linked Telegram so that no catalogued event is silently dropped. Linking a chat identifier for delivery is distinct from Telegram-based authentication, which remains deferred per FR-017.
- **FR-027**: System MUST persist the shopping cart server-side as `Cart`/`CartItem` collections keyed by either an authenticated user or an anonymous cart token (cookie), so the cart syncs across the `undrllanding` storefront and the Telegram Mini App on the shared API contract. Guest checkout MUST be allowed: a guest MUST provide a contact handle (Telegram or email) — required so notifications can be delivered per FR-026 — and MAY optionally create an account after purchase. On login or registration, an anonymous cart MUST merge into the user's cart (merging sums quantities per `product_id` + `variant_id` + `tenant_id` up to available inventory stock caps). The multi-vendor cart's cross-tenant read model is governed by FR-031, and the `anon_token` and guest `contact_handle` MUST meet the entropy, expiry, and anti-abuse requirements of FR-034.
- **FR-028**: System MUST define a single canonical settlement currency per client instance, configured at provisioning; `Product.price` and order totals MUST be denominated in it. Card payments (Stripe) MUST settle in the canonical currency. For crypto providers (TON, SHKeeper), the system MUST convert the canonical amount to the payable crypto amount at checkout using a rate quote with a short TTL that locks that amount for a defined window, and MUST re-quote if the quote expires before payment rather than accepting a stale rate. The `PaymentAttempt` MUST record the settlement currency, quoted rate, crypto amount, and quote expiry.
- **FR-029**: System MUST compute tax at checkout through a common **tax-provider abstraction** (mirroring the payment-provider abstraction). **Stripe Tax** MUST be the engine for card (Stripe) orders. Because Stripe Tax does not price crypto rails, non-Stripe (TON/SHKeeper) orders MUST use a pluggable tax adapter backed by a rates provider, and the system MUST validate EU business VAT numbers via the official **VIES API** to apply B2B reverse-charge (0% when a valid EU business VAT is supplied). Tax collection MUST be gated by configurable nexus/OSS thresholds (e.g. US per-state economic nexus, EU €10k OSS) so tax is collected only where the client is obligated. Computed tax MUST be recorded on the `Order` (`tax_amount`, `tax_mode` inclusive/exclusive, `reverse_charge`, plus a per-line breakdown). Merchant-of-Record engines (Paddle, Lemon Squeezy) are NOT compatible with the multi-vendor Stripe Connect payout model (the MoR becomes the sole seller) and are OUT OF SCOPE for the marketplace configuration.
- **FR-030**: System MUST support a per-`Service` payment policy of **pre-paid** or **pay-later/free**. A pre-paid booking MUST create a linked `PaymentAttempt` at request through the common payment-provider abstraction: for card providers the system MUST authorize on request and capture on confirmation, voiding the authorization if the booking is rejected or cancelled; for crypto providers (no authorize/capture split) the system MUST charge on request and auto-refund on rejection/cancellation via the `initiator=system` FR-020 refund path. Pay-later/free bookings MUST NOT create a payment. A pre-paid booking MUST NOT be confirmed unless its payment captures successfully, and slot reservation on confirmation (FR-019) MUST be atomic with that capture.
- **FR-031**: System MUST define a customer-scoped cross-tenant read model that reconciles the multi-vendor cart (FR-027) with the per-`tenant_id` row filters of FR-003. A customer's own `Cart` and its `CartItem`s (which may reference products across multiple `tenant_id`s) MUST be filtered by the owning customer identity — the authenticated `customer_id` or, for guests, the `anon_token` bound to the cart — and MUST NOT be exposed by any `tenant_id`-scoped vendor policy. A vendor MUST NOT read a customer's full cross-tenant cart; a vendor sees only the `Order`(s) and `OrderSplit` rows carrying its own `tenant_id` after checkout fan-out. Order-split writes MUST fan out one `tenant_id`-scoped `Order` per vendor (FR-024) so that all vendor-facing reads remain `tenant_id`-filtered while the customer retains a unified cross-tenant view of their own carts and orders. Guest order reads require a valid `checkout_secret` issued at checkout. Field permissions MUST prevent a customer read of another customer's cart or order and MUST prevent enumeration of `anon_token`-keyed carts or guest orders.
- **FR-032**: System MUST make webhook and settlement processing resilient beyond dedup/idempotency (FR-013). Inbound provider webhooks MUST be accepted, persisted, and processed asynchronously so a slow downstream never blocks the provider's delivery; transient processing failures MUST be retried with bounded exponential backoff, and a per-provider circuit breaker MUST shed load to a dead-letter/reconciliation queue after a configured failure threshold. The system MUST run a periodic reconciliation sweep that polls provider state for `PaymentAttempt`s stuck in non-terminal states past a timeout. For crypto rails the system MUST model explicit amount-mismatch states on `PaymentAttempt` — `underpaid` (received < quoted, awaiting top-up until `quote_expires_at`), `overpaid` (received > quoted, flag for partial refund via the FR-020 path), and `expired` (no/insufficient funds before `quote_expires_at`, re-quote required) — and MUST NOT mark an order paid unless the received amount satisfies the quoted amount within a configured tolerance.
- **FR-033**: System MUST make notification delivery (FR-026) resilient. A failed delivery on the primary Telegram channel MUST be retried with bounded exponential backoff before the email fallback is attempted, and a failed email fallback MUST likewise be retried with backoff; each attempt MUST be recorded on `NotificationDelivery` (`attempt_count`, `last_error`, `next_retry_at`, terminal `status`) so no catalogued event is silently dropped and repeated retries do not duplicate a successful delivery.
- **FR-034**: System MUST secure guest identifiers and contact handles against hijack, spam, and enumeration. The anonymous cart `anon_token` MUST be generated with cryptographic entropy (>= 128 bits from a CSPRNG), stored/compared as an opaque high-entropy secret, set in an `httpOnly`, `Secure`, `SameSite` cookie, and carry a bounded expiry (idle + absolute TTL) after which it is invalidated; a token MUST NOT be guessable or enumerable and MUST rotate on account merge (FR-027). Guest `contact_handle` values used for notifications MUST be rate-limited per handle and per source to prevent notification spam, MUST be verified (Telegram deep-link linkage or an email verification/confirmation step) before high-volume or sensitive notifications are sent, and MUST NOT allow account/handle enumeration through differing responses or timing.
- **FR-035**: System MUST define the storage and injection model for per-instance secrets used by FR-013 and FR-025 — Stripe secret/publishable keys, Stripe webhook signing secrets, TON credentials, and SHKeeper API keys/webhook secrets. Secrets MUST be provisioned per isolated instance (never shared across client instances or committed to `schema.snapshot.yaml` or source), injected at deploy time via the `undevops` secret store / environment (FR-022) rather than baked into images, referenced by the Directus extensions through environment/secret references only, and MUST be rotatable without a code change. The provisioning runbook MUST document generation, injection, rotation, and revocation of these secrets.
- **FR-036**: System MUST treat delivery addresses, guest contact handles, Telegram chat identifiers, VAT numbers/VIES responses, and payout wallet addresses as sensitive operational PII. These fields MUST use structured validation at API boundaries, role-specific field permissions/masks, audit-log redaction, and documented retention/deletion rules. Vendor reads MUST be limited to the PII needed to fulfil their own order or booking; customers MUST NOT read another customer's contact/address/tax data, and logs MUST NOT contain raw VAT numbers, contact handles, or full delivery addresses.

### Key Entities

- **Tenant**: Represents a sub-organization or storefront within one client-owned Directus instance (`id`, `name`, `domain`, `branding_config`). Client marketplaces themselves are isolated Directus instances/containers, not `Tenant` records.
- **Product**: Physical or digital goods offered for sale (`id`, `tenant_id`, `title`, `price`, `type`, `inventory_quantity`, `custom_fields`).
- **InventoryReservation**: A time-limited checkout hold for physical inventory (`id`, `order_id`, `product_id`, `quantity`, `status`, `expires_at`).
- **Service**: Consultations or bookable services (`id`, `tenant_id`, `provider_id`, `hourly_rate`, `price`, `payment_policy`, `availability`).
- **Booking**: A request or reservation for a service slot (`id`, `tenant_id`, `service_id`, `provider_id`, `customer_id`, `starts_at`, `ends_at`, `status`, `payment_attempt_id`, `rescheduled_from_id`).
- **Cart**: Server-side shopping cart keyed by user or anonymous token (`id`, `customer_id`, `anon_token`, `anon_token_expires_at`, `contact_handle`, `contact_verified`, `updated_at`). Customer-scoped and cross-tenant per FR-031; `anon_token` is a high-entropy secret per FR-034.
- **CartItem**: A line within a cart (`id`, `cart_id`, `product_id`, `tenant_id`, `quantity`).
- **Order**: Single-vendor customer purchasing record (`id`, `tenant_id`, `checkout_group_id`, `customer_id` (nullable for guest), `customer_contact`, `items`, `total_amount`, `platform_fee_amount`, `tax_amount`, `tax_mode`, `reverse_charge`, `status`, `payment_method`, `fulfilment_status`, `tracking_number`, `delivery_method`, `delivery_address`, `shipping_amount`). A multi-vendor cart yields one `Order` per vendor sharing a `checkout_group_id`.
- **PaymentAttempt**: Provider-agnostic payment audit and state record (`id`, `order_id`, `provider`, `status`, `settlement_currency`, `quoted_rate`, `crypto_amount`, `received_amount`, `amount_mismatch_state`, `quote_expires_at`, `idempotency_key`, `provider_payment_id`, `provider_event_id`, `refund_amount`). `amount_mismatch_state` (`ok`/`underpaid`/`overpaid`/`expired`) captures the crypto reconciliation states of FR-032.
- **OrderSplit**: The per-vendor fan-out record linking a `checkout_group_id` to each single-vendor `Order` and its `PaymentAttempt` (`id`, `checkout_group_id`, `tenant_id`, `order_id`, `payment_attempt_id`, `platform_fee_amount`). Vendor-facing rows are `tenant_id`-filtered per FR-031 while the customer reads the whole group.
- **VendorPayoutAccount**: Per-provider payout configuration for a vendor (`id`, `tenant_id`, `provider`, `status`, `stripe_connect_account_id`, `payout_wallet_address`, `onboarding_completed_at`).
- **TelegramLink**: Binds a Directus user to a Telegram chat for bot notification delivery (`id`, `user_id`, `telegram_chat_id`, `linked_at`).
- **NotificationDelivery**: An outbound notification audit record (`id`, `event_type`, `recipient_user_id`, `recipient_role`, `channel`, `status`, `attempt_count`, `last_error`, `next_retry_at`, `sent_at`). Retry/backoff semantics per FR-033.
- **RefundRequest**: A customer-initiated full or partial refund request (`id`, `order_id`, `payment_attempt_id`, `requested_amount`, `approved_amount`, `status`, `reason`, `approved_by`).
- **DeliveryZone**: A vendor-configured geographic delivery area and rate (`id`, `tenant_id`, `name`, `address_match_rules`, `rate_type`, `rate_amount`, `free_from_amount`).
- **TaxRegistration**: A jurisdiction where the instance is registered/obligated to collect tax, with thresholds (`id`, `jurisdiction`, `tax_provider`, `collect_enabled`, `nexus_type`, `threshold_amount`).
- **DigitalEntitlement**: Purchaser-scoped permission to download a paid digital product (`id`, `order_id`, `product_id`, `customer_id`, `status`, `expires_at`, `revoked_at`).
- **BrandingSettings**: Visual tokens mapped from `@underundre/undesign` (`tenant_id`, `primary_color`, `secondary_color`, `theme_name`, `logo_url`).
- **MutualAidQueue** (Flagship): Queue records for critical aid recipients (`user_id`, `goal_amount`, `collected_amount`, `need_category`, `proof_urls`, `flow_score`, `position`).
- **VpnNode** (Flagship): Registered `unet` VPN instances (`id`, `owner_id`, `public_ip`, `awg_pubkey`, `uptime_seconds`, `flow_reward`).
- **BarterItem**: P2P exchange listings (`id`, `owner_id`, `title`, `description`, `desired_exchange`, `listing_fee_status`).
- **TaxiRequest** / **CourierJob** (Flagship, deferred FR-008): Logistics job (`id`, `role` taxi|courier, `requester_id`, `origin_geo`, `destination_geo`, `offered_price`, `status`, `accepted_worker_id`).
- **WorkerProfile** (Flagship, deferred FR-008/038/039): Worker registration (`id`, `user_id`, `roles[]` courier|taxi_driver|warehouse_owner|housing_pro, `service_area`, `base_rate`, `status`).
- **WarehouseListing** (Flagship, deferred FR-038): Storage offer (`id`, `owner_id`, `location`, `capacity`, `price`, `currency`, `status`).
- **MarketplaceDirectoryEntry** (Flagship, deferred FR-037): Hub registration for an opt-in client marketplace (`id`, `instance_ref`, `display_name`, `public_url`, `category_tags`, `listed`, `catalog_sync_status`, `description`).
- **FederatedCatalogItem** (Flagship, deferred FR-037): Searchable product/service summary synced from a client instance (`id`, `source_instance_id`, `source_item_type` product|service, `source_item_id`, `title`, `price`, `currency`, `category`, `media_url`, `deep_link`, `indexed_at`, `available`).
- **HubCart** / **HubCartItem** (Flagship, deferred FR-037): Unified cart across instances (`HubCart`: id, customer_id|anon_token; `HubCartItem`: source_instance_id, source_item_type, source_item_id, quantity, price_snapshot, currency, vendor/tenant ref).
- **HubCheckoutGroup** (Flagship, deferred FR-037): Fan-out parent linking per-source-instance (and per-vendor) settlement legs (`id`, `checkout_group_id`, legs[], aggregate_status).

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Clean client marketplace instances can be fully bootstrapped from `schema.snapshot.yaml` via `undevops` in under 60 seconds.
- **SC-002**: 100% of API endpoints strictly respect Directus policy/permission row filters and field permissions, verified via automated test suites without data leakage across tenants.
- **SC-003**: Frontend theme changes via `@underundre/undesign` render dynamically on storefronts without requiring application rebuilds.
- **SC-006**: The `undrllanding` Next.js storefront and Telegram Mini App each complete catalog, cart, checkout, and service-request journeys against the same versioned API contract in automated end-to-end tests.
- **SC-007**: Automated tests verify that physical checkout atomically reserves the final available unit, only confirmed payment consumes that reservation, cancelled/failed/expired payment releases it, manual fulfilment can record a tracking number, and a digital download is inaccessible before confirmed payment or after entitlement revocation.
- **SC-008**: Automated end-to-end tests verify that customers can request an available service slot, providers can confirm or reject it, and cancellation or rescheduling updates the booking state and slot availability in both storefront clients.
- **SC-009**: Concurrent booking tests verify that confirming one of two overlapping pending requests succeeds exactly once and transitions every competing request to `rejected` without double booking.
- **SC-010**: Automated tests verify that a vendor or administrator can approve a full or partial refund no greater than the captured amount, rejected requests do not alter payment state, and only a provider-confirmed refund updates the order.
- **SC-011**: Automated checkout tests verify that delivery requires an address and applies only the matching configured zone rate, while local pickup requires neither; addresses outside all zones cannot complete delivery checkout.
- **SC-012**: Automated tests verify that each catalogued notification event is delivered to the correct role via the Telegram bot when linked and via email fallback when not, with one recorded `NotificationDelivery` per event and no silent drops.
- **SC-013**: Automated tests verify that an anonymous server-side cart persists and merges into the user cart on login, and that a guest can complete checkout with a contact handle and receive notifications without creating an account.
- **SC-014**: Automated tests verify that catalog prices and order totals use the instance canonical currency, and that a crypto checkout locks a quoted amount for its TTL, re-quotes on expiry, and records the rate/amount on the `PaymentAttempt`.
- **SC-015**: Automated tests verify that tax is computed at checkout via the tax-provider abstraction (Stripe Tax for card, adapter for crypto), that a VIES-valid EU business VAT applies reverse-charge, that collection respects configured nexus/OSS thresholds, and that computed tax is recorded on the `Order`.
- **SC-016**: Automated tests verify that a pre-paid booking creates a linked `PaymentAttempt`, that capture and slot reservation on confirmation are atomic, that rejection voids the card authorization or auto-refunds the crypto charge, and that pay-later/free services create no payment.
- **SC-004**: `unet` client daemon establishes DPI-resistant AmneziaWG user-space tunnels on Windows/macOS/Linux without root/admin privilege prompts. *(Deferred to the follow-up flagship spec; not a gating criterion for `001-init`.)*
- **SC-005**: Mutual Aid Queue scoring accurately recalculates user positions based on verified activity metrics within 5 seconds of event ingestion. *(Deferred to the follow-up flagship spec; not a gating criterion for `001-init`.)*

---
*End of Specification.*
