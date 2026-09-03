# Undrlla — flows & gap audit

**Date**: 2026-07-22  
**Scope**: Whole product surface as of `001` + `002` + `003` + roadmap decisions (hub unified cart, logistics, housing+CBI).  
**Legend**: ✅ thought through in specs · 🟡 partial / deferred with intent · 🔴 gap (undecided or contradictory) · ⚫ out of scope by design

> ## ⚠ STALE BANNER (2026-07-30)
>
> This audit is **pre–Medusa / pre–Paddle pivot** in large parts. Do **not** drive sequencing or implement from 🔴 rows alone.
>
> | After this audit | Source of truth |
> |------------------|-----------------|
> | Client shops | **Medusa** (`undreseller`) — not Directus Tier-A commerce |
> | Fiat pre-LLC | **Paddle MoR** — Stripe post-LLC |
> | SSO / JWT | Locked in `005-sso-jwt-contract.md` |
> | Pricing / SaaS fee | Locked in `008-pricing-and-affiliate.md` |
> | Launch sequence | `ECOSYSTEM-REQUIREMENTS.md` §5 Phase A–D |
> | Housing KYC MVP | Manual/agency (`003` FR-012) |
>
> **Action:** re-run FLOWS-AUDIT v2 after Phase A (template-shop + provision + storefront) green. Until then prefer ECOSYSTEM + ECOSYSTEM-REQUIREMENTS.

---

## 0. Actors

| Actor | Wants |
|-------|--------|
| **End customer** | Buy goods/services on client shop or hub; book services; ride/courier |
| **Donor** | Fund housing/CBI (pool or person) |
| **Housing applicant** | Raise / receive funded housing+CBI package |
| **Client merchant (store owner)** | Get a shop, sell, optionally list on hub, hire courier/warehouse |
| **Vendor (in a shop)** | Sell products/services, get paid |
| **Worker** | Courier / taxi / warehouse / housing pro — get jobs, get paid |
| **Founder/concierge (MVP)** | Deliver housing escort services |
| **Platform admin / undevops op** | Provision, moderate, KYC gates, pool allocate |
| **Notary / escrow agent** | Receive housing funds, confirm signing |

---

## 1. Flow map (happy paths)

### A. Client marketplace lifecycle

```
Request provision (002) → manifest validate → undevops deploy (001)
  → domain/TLS → seed admin → theme/branding
  → admin invites vendors (001 v1) → payout setup Connect/crypto
  → products/services listed → customer browses storefront or Mini App
  → cart → checkout fan-out per vendor → pay → fulfil / book
  → [opt-in] share_catalog_to_hub → appears in hub index
```

| Step | Status | Notes |
|------|--------|--------|
| Operator provision | ✅ `002` | Customer self-service portal still weak |
| Domain BYO / subdomain | ✅ `001` FR-023 | |
| Secrets injection | ✅ FR-035 | |
| Multi-vendor cart/checkout | ✅ FR-024/027/031 | |
| Payments + webhooks | ✅ FR-013/032 | |
| Refunds, tax, currency | ✅ FR-020/028/029 | |
| Notifications TG+email | ✅ FR-026/033 | |
| Hub opt-in flags | 🟡 `002` FR-011 | No full `004` yet |
| SaaS billing for instance | 🔴 | Who pays monthly for the shop? |
| Customer self-service provision | 🟡 deferred | |

### B. Hub: search + unified cart (target v1 of `004`)

```
Client opts in → catalog sync → FederatedCatalogItem index
  → user searches on Undrlla.com → add HubCart lines (multi-instance)
  → revalidate stock/price → checkout fan-out legs
  → pay each leg → orders land on source instances → fulfil
```

| Step | Status | Notes |
|------|--------|--------|
| Index + search intent | 🟡 FR-037 | No dedicated spec yet |
| Unified cart | 🟡 FR-037 d–h | Locked as v1, not designed deep |
| SSO / one identity across instances | 🔴 | Critical for hub |
| Where order “lives” after pay | 🔴 | Hub ledger vs write-through to client Directus |
| Service booking in unified cart | 🔴 | Slot + multi-leg = hard |
| Display currency strategy | 🔴 | Listed open |
| Cart merge hub ↔ client | 🔴 | Listed open |
| Client revokes hub share mid-cart | 🔴 | |
| Partial pay UX (leg A ok, B fail) | 🟡 | FR-024 spirit, no hub UX |

### C. Housing + CBI (003)

```
Admin enables ServiceArea TR/GE → HousingProgram housing_cbi $500k
  → applicant submits HousingApplication + requested_amount
  → ≤500k auto amount OK; >500k admin amount-approve
  → KYC verified → (pool funded somehow) → allocate full amount
  → escrow to notary snapshot → signing confirmed → release
  → optional concierge bookings (founder MVP)
```

| Step | Status | Notes |
|------|--------|--------|
| Program + amount rules | ✅ | |
| Escrow never cash to applicant | ✅ | |
| Country gate | ✅ | |
| Concierge as Service | ✅ | founder-only MVP |
| Multi-vendor lawyers/realtors | 🟡 post-MVP FR-019 | self-signup locked |
| **How donors fund a specific person** | 🔴 | Pool is shared; application has target but earmark unclear |
| **Donation → HousingPool credit path** | 🟡 | Event hook mentioned; product UX not |
| Progress bar / public campaign page | 🔴 | |
| Donation refunds / chargebacks | 🔴 | |
| Donor SoF AML for large gifts | 🟡 | flag only, no provider |
| Model B self-fund / Model C self-pay | ⚫ deferred | Users will ask day 1 |
| Partial pool allocation | ⚫ out MVP | Queue starvation risk |
| CBI legal min hard block | 🟡 warn-only | |
| 3-year hold tracker | 🔴 | |
| Post-purchase rental PM | ⚫ post-003 | |
| Platform fee on donations | 🔴 | not locked (vs 12% jobs) |

### D. Logistics (005 intent)

```
Worker self-signup → Connect/crypto → set area + price
  → requester posts job (hub or client “deliver this order”)
  → workers see board (price sort) → accept → complete → 12% fee
```

| Step | Status | Notes |
|------|--------|--------|
| Roles + fee 12% | 🟡 intent | no state machine |
| Pay when? (prepay / on complete / escrow) | 🔴 | |
| No-show / cancel / dispute | 🔴 | |
| PII: customer address → courier | 🔴 | |
| Link job ↔ Order id | 🔴 | |
| Warehouse capacity double-book | 🔴 | |
| Reviews | ⚫ later | |
| Insurance | 🔴 open | |
| Geo matching algorithm | 🔴 | “area” freeform |

### E. Identity & access (cross-cutting)

```
Email/password per instance (001) · TG link for notify · TG auth deferred
Workers self-signup (flagship) · Shop vendors admin-invite (001 v1) · Conflict
Hub user ≠ automatically client-instance user
```

| Topic | Status |
|-------|--------|
| Per-instance auth | ✅ |
| Cross-instance SSO for hub | 🔴 |
| Self-signup vs invite vendors | 🔴 **contradiction** |
| Roles matrix (customer+worker+donor+applicant) | 🔴 |
| Admin RBAC platform vs client | 🟡 |

### F. Money matrix

| Flow | Who pays | Who gets | Platform cut | Spec |
|------|----------|----------|--------------|------|
| Shop product | Customer | Vendor | % FR-025 | ✅ |
| Service booking prepaid | Customer | Provider | % | ✅ |
| Logistics job | Requester | Worker | **12%** | 🟡 |
| Housing donation | Donor | Pool → notary | **?** | 🔴 |
| Housing Model C | Buyer | Notary | commission deferred | ⚫ |
| Instance SaaS | Merchant | Platform | subscription | 🔴 |
| Barter listing fee | User | Platform | flat | 🟡 FR-007 |
| MAQ $1/mo | Donor | Pool | ? | 🟡 |

### G. MAQ / VPN / Barter / AI

High-level in `001` User Story 3 only. No end-to-end product flows. Treat as **horizon**, not MVP chain.

---

## 2. Critical gaps (blocking coherent product)

### G1 — Identity federation (hub)
Without SSO or account linking, unified cart cannot cleanly own multi-instance orders, refunds, or “my purchases”.  
**Need**: flagship account + link/claim to client instances, or hub-as-MoR with pass-through (legal heavy).

### G2 — Vendor signup contradiction
- `001` FR-025: vendors **admin-invited only**  
- Workers / housing pros: **self-signup**  
- Client shops that want open marketplace: unclear  
**Need**: explicit policy per context (client Tier-A invite-only v1; flagship workforce self-signup + KYC gate).

### G3 — Housing: pool vs personal campaign — **CLOSED 2026-07-22**
**Locked: shared pot + queue (B).** Donors subscribe/top-up `HousingPool`; platform/queue picks next verified applicant. `requested_amount` = need size when funded, not a donor-facing thermometer. Earmarked peer donations **post-MVP**.

### G4 — Donation economics — **CLOSED 2026-07-22 (fee)**
**Locked: 5%** platform fee on donations + pass-through payment fees (default). Still open: chargeback/refund policy detail for subscriptions.

### G5 — Hub order write path — **CLOSED direction: Hub MoR**
Customer pays **flagship Undrlla (MoR)**; settle net to vendors via Connect/crypto; fulfilment stays with source vendor. Implementation detail (sync shape) still for `004`.

### G6 — Logistics payment lifecycle — **CLOSED 2026-07-22**
**Hold/authorize on accept → capture on complete**; void on cancel; dispute blocks capture. Crypto escrow-equivalent documented per rail.

### G7 — Dual delivery models
`001` FR-021: vendor delivery zones on the shop.  
`001` FR-038: flagship courier board.  
**Need**: when checkout offers zone rate vs “hire Undrlla courier”; avoid double charge / no delivery.

### G8 — Services in unified cart
Products are easy; **bookable services** need slot hold across revalidation + multi-leg pay. Undesigned.

### G9 — Instance commercial model
Who pays for hosting/support of client shops? `002` has billing_reference but no subscription product. Platform revenue otherwise is only take-rates.

### G10 — Compliance packaging
CBI/housing AML, worker employment vs contractor classification, marketplace payments licensing, hub multi-jurisdiction tax — flagged piecemeal, no single compliance plan.

---

## 3. Secondary gaps (painful but sequenceable)

| ID | Gap |
|----|-----|
| S1 | Reviews/ratings (workers, vendors, housing pros) |
| S2 | Dispute center (orders, jobs, donations) |
| S3 | 3-year CBI hold reminders |
| S4 | Partial allocation / waitlist fairness for housing |
| S5 | Model B (save for own housing) — high user demand |
| S6 | Chat between parties (barter mentions undrepost; logistics/housing need it) |
| S7 | Catalog sync lag / tombstones when product deleted |
| S8 | Hub moderation (scam shops, fake CBI offers) |
| S9 | Worker geo: structured areas vs free text |
| S10 | Merge guest hub cart with client carts |
| S11 | Multi-currency display choice |
| S12 | Pressure Points: which activities count (jobs? housing help?) |
| S13 | `003` reviews gate vs implement readiness |
| S14 | Client store “create courier job” API contract with flagship |

---

## 4. What is actually solid

1. **Tier-A shop core** (products, multi-vendor cart, payments, bookings, refunds, tax, PII, notifications) — deep and consistent.  
2. **Isolation model** (instance per client + policies inside) — clear.  
3. **Housing escrow principle** (never raw cash to applicant; destination snapshot) — strong.  
4. **Amount self-serve ≤500k / over-cap approve** — clear.  
5. **ServiceArea geo feature flags** — extensible.  
6. **Roadmap product bets** locked: federated hub+cart, logistics pool, 12% jobs, self-signup workers.

---

## 5. Suggested design order (close gaps without boiling ocean)

| Priority | Work | Closes |
|----------|------|--------|
| P0 | Decide housing **A/B/C funding model** (earmark vs pool) + donation fee % | G3, G4 |
| P0 | Identity: flagship account + instance link strategy for hub | G1 |
| P0 | Logistics job state machine + pay/capture/dispute | G6 |
| P1 | Hub order write-through design + services-in-cart rules | G5, G8 |
| P1 | Vendor/worker signup policy matrix | G2 |
| P1 | Delivery zone vs Undrlla courier at checkout | G7 |
| P2 | Instance SaaS billing | G9 |
| P2 | Compliance checklist doc | G10 |
| P3 | Reviews, chat, CBI hold, Model B | S* |

---

## 6. Verdict

**Not everything is thought through.**  
Tier-A marketplace + housing escrow skeleton are strong.  
Flagship “super-app” (hub cart, logistics, personal CBI fundraising, multi-role identity) is **directionally decided** but has **structural holes** (identity, housing money model, job payment lifecycle, hub order ownership).  

Shipping order that won’t explode:

1. `001` core shop  
2. `002` provision  
3. `003` housing **after** locking G3+G4  
4. `005` logistics simple  
5. `004` hub unified cart (hardest; needs G1+G5 first)

---

*End of audit.*
