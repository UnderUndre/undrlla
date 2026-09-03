# Undrlla product roadmap (planning index)

Living index of platform intent across SpecKit features. Product **values / person tiers / red lines**: [`CONSTITUTION.md`](./CONSTITUTION.md). Cross-repo readiness + launch gates: [`ECOSYSTEM-REQUIREMENTS.md`](./ECOSYSTEM-REQUIREMENTS.md). Feature folders remain source of truth for pipe detail when not superseded by ECOSYSTEM Q8–Q11.

**Updated**: 2026-08-06

---

## 🎯 Immediate Focus & Execution Horizon

### 🟢 NOW (30 Days) — Phase A Commerce Core & Book Launch (Single-Pipe Focus)

#### **NOW Priority 0 (P0 — Must Not Slip)**:
1. **`undreseller` Runnable Template-Shop (`apps/template-shop`)**: Complete local Docker Compose stack (Medusa 2.0 + Postgres + Redis) with integrated **Paddle Sandbox** checkout.
2. **Open-Source Book Release (`undreading`)**: Publish *"Сантехника бытия"* manuscript (v3.2) on GitHub / Web / Notion ($0 Open Access) + Chapter 1 lead magnet & printable checklists.
3. **Direct Telegram Sales (`@undreading_bot` / `undrepay`)**: 1-click payment for Digital Pack (**$13.37** LEET) & **Founding Citizen Pass ($69.00** "Nice"; 100 passes = $6,900 relocation capital).

#### **NOW Priority 1 (P1 — Execution Flex 7–10 Days)**:
1. **`undevops` Medusa Provisioning Job**: Ingest `ProvisioningManifest` to deploy client Medusa containers + Traefik + ACME TLS with pre-flight A/CNAME DNS check.
2. **First Client Deployment (Founder Bootstrap)**: Founder executes first 1–2 client setups directly using the template pipeline (Mode A bootstrap).
3. **Amazon KDP Launch**: Kindle eBook (**$6.70** "6-7 Aura" meme price -> $4.69 net to Payoneer) & Paperback print-on-demand ($19.99).

#### **NOW Priority 2 (P2 — Multi-Platform Campaign)**:
1. Build in Public campaign across Telegram, X/Twitter, Habr, and Hacker News ("Show HN").

### 🟡 NEXT (60–90 Days) — Growth, Guild Onboarding & Book Virality
1. **Dev Guild Onboarding (`007-undrecruiting` Mode A)**: Onboard first external developers; dispatch store setups via `undrepay` escrow hold (70% dev / 30% founder split upon `/code_review` PASS).
2. **Flagship IdP & SSO (`005-sso-jwt-contract.md`)**: Directus 12.x RS256 JWKS endpoint + `@medusajs/auth` federation across `.undrlla.network`.
3. **Housing Pool Seed (`003-housing-donations`)**: Shared donation pot ($1/mo subscription) + manual/agency KYC intake for **Cyprus (CY - €300k Fast-Track PR)**, **UAE (AE - 2M AED Golden Visa)**, **Panama (PA - $300k QIV)**, **Greece (GR - €250k conversion)**, **Malta (MT - €375k MPRP)**, and Primary Home targets **Canada Alberta (CA-AB)** & **US North Carolina/Washington (US-NC/WA)** (**Georgia GE = 1% Small Business tax setup only, NOT housing purchase; Turkey TR = advisory warning for SPK appraisal gap/seismic fault; US-TX/FL & ES/PT/DE = warning/blacklist gates**).
4. **Book Virality & Affiliate Engine**:
   - Send Advance Reader Copies (ARC) to 30+ tech & longevity influencers for blurbs/reviews.
   - Launch 20% affiliate RevShare for newsletter & podcast partners referring book/pass buyers.

### 🔵 LATER (Post-Revenue / 6+ Months) — Expansion
1. **`undrepay` Crypto Processing**: Blnk double-entry ledger + SHKeeper crypto intents (`upi_...`).
2. **Marketplace Hub Search & Catalog (`001` FR-037)**: Federated catalog indexing (search first, MoR charges after legal/tax audit).
3. **Logistics Workforce & Dedicated Crew (`005-logistics-workforce` / `007`)**: Simple courier/taxi/warehouse match board (12% job fee) + Dedicated Subscription Drivers/Couriers (quota + overage billing + language matching).
4. **Twenty CRM Integration (`twentyhq/twenty`)**: Self-hosted Open-Source CRM container provisioned via `undevops` for enterprise B2B sales pipeline, AI Hunter leads, and deal tracking.
5. **Autonomous P2P Bounties (`007` Mode B)**: Automated GitHub Issue bounties via Opire/Algora pattern.

---

## Platform shape

| Layer | What | Spec |
|-------|------|------|
| **Constitution** | Digital-polity principles, citizen/migrant, handles, mobility red lines | `CONSTITUTION.md` (v0.1) |
| **Marketplace template** | Clean multi-vendor shop (Medusa monorepo fork) | **`undreseller`** (`develop`, see `UNDERUNRE.md`) — **source of truth for client shops** |
| **Client Store Service & Dev Guild** | Custom store delivery via Dev Guild + `undrepay` escrow hold | `007-undrecruiting.md` + `plans/qwen-3.8.md` |
| **Tier A core (legacy)** | Directus marketplace in undrlla | `001-init-from-directus` — **do not dual-build for clients**; hardened LEGACY BANNER 2026-07-29 |
| **Provision** | Create/deploy isolated client Medusa instances + domains | `002-provisioning-portal` + **`contracts/medusa-provisioning-manifest.schema.json`** + `undevops/003` |
| **SSO / JWT** | Zero-code IdP + Medusa auth federation | `005-sso-jwt-contract.md` |
| **Pricing / Alpha 50** | $29/mo = `undreseller` template + `undevops` hosting + 1 shop deploy + citizen credit | `008-pricing-and-affiliate.md` |
| **Housing KYC (MVP)** | Manual admin / contracted agency — no mandatory Sumsub/Onfido | `003` FR-012 |
| **Hub (federated catalog + unified cart)** | Index + search + **one cart across client shops** + checkout fan-out to vendors | `001` FR-037 *(deferred)*; `002` FR-011 opt-in |
| **Housing + path-to-papers** | Donation-funded housing; papers track country **may differ**; founder concierge MVP | `003-housing-donations` (+ FR-025 housing⟂papers) |
| **Identity handles** | Global `@user` + hub + TM reclaim | `004-identity-handles.md` (outline) |
| **Union matching** | Serious dating + logistics; no passport guarantee | `006-union-matching.md` (outline) |
| **undrecruiting** | IT + housing crew proof-hire + Dev Guild Bounties | `007-undrecruiting.md` |
| **Citizen pricing** | Discount table draft | `citizen-pricing.md` |
| **Payments** | Fiat now = **Paddle MoR**; later = **Stripe** (post US LLC); crypto = **undrepay** | ECOSYSTEM Q10 · `Paddle-into-JS-TS-web-app.md` · `../undreseller/specs/TEMPLATE-SHOP-PLAN.md` |
| **Template-shop** | Deployable Medusa store + Paddle (SpecKit ready) | `../undreseller/specs/001-template-shop/` (spec/plan/tasks) |
| **Logistics work** | Couriers + taxi + warehouse; self-signup; Stripe/crypto payouts; **12%** default job fee; shared flagship pool, jobs from hub or client store | `001` FR-008, FR-038 *(deferred; suggested `005-logistics-workforce`)* |
| **Housing workforce** | Lawyers, realtors, translators, escorts; self-signup post-MVP; ordinary platform fee | `001` FR-039; `003` FR-019 post-MVP |
| **MAQ / VPN / Barter / AI builder** | Flagship extras | `001` FR-007, FR-009–FR-012 *(deferred)* |
| **Rental PM + smart home** | After purchase: rent in-app, optional locks | `003` FR-018 *(post-003)* |

## Client Store Assembly & Dev Guild Pipeline (Escrow Delivery)

To scale store delivery without overloading founder time:

1. **Client Brief & Upfront Escrow**: Client orders a store setup ($500–$2,500). Payment (fiat via Paddle or crypto via `undrepay`) is held in an automated `undrepay` **Escrow Hold** (`upi_...` intent).
2. **Template Assembly (No Re-inventing)**: Stores are deployed using the `undreseller` Medusa template + `undevops` 1-click pipeline (0 code from scratch).
3. **Dev Guild Execution (`007-undrecruiting`)**:
   - **Phase 1 Bootstrap (Founder-as-Dev)**: Founder executes initial client setups to battle-test the assembly line and capture 100% margin.
   - **Phase 2 Scale (Mode A Managed Agency)**: Founder acts as Chief Architect (spec + code review), assigned Guild developer executes theme/customizations. Payout split: 70% developer, 30% founder/platform upon code review PASS.
   - **Phase 3 Autonomous (Mode B P2P Bounties)**: Custom tasks attached to GitHub issues; Guild devs submit PRs. Automated test pass + peer review unlocks `undrepay` escrow (5–10% protocol fee).
4. **Verified Escrow Release**: Client payment is captured from `undrepay` escrow and disbursed to developer ONLY when `undevops` deploy passes healthcheck and client/SLA sign-off is confirmed.

## Value Proposition: Alpha 50 Subscription ($29/mo)

Subscriber receiving $29/mo Alpha 50 membership gets:
1. **PaaS Hosting & Template License**: 1 provisioned client store on `undevops` + access to `undreseller` template updates.
2. **Dev Guild Dispatch**: Priority dispatch of Dev Guild developers for custom store builds at discounted platform rates.
3. **Polity Citizenship Credit**: Qualifying monthly billing count toward `undrlla.citizen` status + $1 contribution to the shared `HousingPool`.

## Simple-first logistics (intent)

1. Worker **self-signup** + role + area + price + Stripe Connect / crypto payout setup.
2. Job created from **Undrlla.com** or from a **client shop** (deliver this order) → same **flagship** worker pool.
3. Select by **price** (reviews later).
4. Accept → complete → platform fee **12%** default (configurable 10–15%, optional min fee).
5. No full Uber/WMS in v1.

## Hub search + unified cart (intent, v1)

1. Client opts in at provision (`share_catalog_to_hub`).
2. Sync product/service summaries → flagship index.
3. User searches on Undrlla.com → **add to unified cart** (lines from many instances).
4. Checkout **fan-out**: one payment/order leg per source instance (and vendor); shared `checkout_group_id`; mixed outcomes OK.
5. Revalidate stock/price at add-to-cart and checkout; tax/currency per leg.

## Suggested next feature slugs (not started)

| Slug | Purpose |
|------|---------|
| `undreseller` bootstrap | Re-seed Medusa tree; undevops provision contract |
| `004-identity-handles` | Promote `004-identity-handles.md` → full SpecKit when scheduled |
| `004-marketplace-hub` | Federated catalog + unified cart *(slug collision with handles — rename when scheduled)* |
| `005-logistics-workforce` | Courier/taxi/warehouse simple v1 + self-signup + Connect payouts |
| `006-union-matching` | Promote `006-union-matching.md` after housing MVP |
| `006-rental-pm` | Leases, rent, owner split *(collision — rename when scheduled)* |
| `007-undrecruiting` | Promote `007-undrecruiting.md` |
| `00x-reviews` | Shared ratings for services + logistics workers |

## Decisions locked (2026-07-22)

| Topic | Decision |
|-------|----------|
| Flagship hero story | **Housing + path-to-papers** funded by community; housing country **may ≠** papers country (2026-07-27) |
| Housing donor model | **Shared pot + queue**; donors subscribe/top-up pool; **no earmark** in MVP (earmark later) |
| Subscription amounts | **Custom ≥ $1** (no fixed grid); one-offs ≥ $1 |
| Queue formula | **Model B:** FIFO; **active sub ≥$1/mo keeps seat**; key = max(first contrib, verified, sub activated) |
| Queue pause | Failed pay → 7d → **paused** (keep key) → **90d** → left (rejoin = new key) |
| “Neediness” | **No score** — only **verified** |
| Sub period | **Monthly**; custom ≥ $1 |
| Donation refunds | **No** self-serve refund after success; legal/chargeback exceptions only |
| Donation platform fee | **5%** + pass-through payment fees (default) |
| Applicant need size | Default **$500k**; ≤500k self-serve amount; >500k admin |
| Hub depth | Search + **unified cart** |
| Hub commerce model | **Hub Merchant of Record** (customer pays Undrlla → settle to vendors) |
| Logistics entry | Flagship board + client-store job → shared pool |
| Worker payouts | Stripe Connect + crypto |
| Worker signup | Self-signup |
| Job platform fee | **12%** default |
| Logistics pay lifecycle | **Hold on accept → capture on complete**; void on cancel; dispute blocks capture |

## Spec hygiene (2026-07-29)

| Done | Item |
|------|------|
| ✅ | `001` LEGACY BANNER hardened (Medusa-only client shops) |
| ✅ | `002` Medusa ProvisioningManifest schema + example |
| ✅ | `005` SSO Zero-Code contract |
| ✅ | `008` pricing + affiliate + hub economics |
| ✅ | `003` KYC MVP = manual/agency |
| ✅ | undrllanding `001` + undevops `003` retargeted to Medusa |
| ✅ | undreseller `001-template-shop` SpecKit + T006 manifest align |
| ✅ | undevops `003` plan/tasks Medusa-only |
| ✅ | undrepay `tasks.md` + StripeBlnkProxy DEPRECATED banner |
| ✅ | citizen-pricing aligned to `008` ($29 SaaS, 0% GMV) |
| ⬜ | undreseller runnable template-shop **code** (Phase A) |
| ⬜ | undevops Medusa ingest job **code** |
| ⬜ | undrepay tracer **code** |

## Still open

- Hard-enforce CBI/papers advisory minima vs warn-only (`003`).
- Billing period for custom sub (monthly default? weekly?).
- Grace period length for lapsed sub (default 1 cycle).
- Display currency strategy on hub.
- Logistics dispute SLA / insurance product.
- Citizen discount table lock (`citizen-pricing.md` is draft; unet list TBD).
- Hub cart merge with native client-instance carts.
- **Hub MoR vs marketplace (non-MoR)** — depends on legal/ops (see below).
- undrepay INTEGRATION O1–O4 founder ACK.

### Closed (hygiene 2026-07-30)

- **undreseller vs Directus Tier-A for client shops** — **kill dual path**; Medusa only (banner + `001` tasks `do-not-implement`).
- **SSO cookie/JWT** — `005-sso-jwt-contract.md`.
- **Housing pool fiat rail label** — Paddle pre-LLC / Stripe post-LLC (not bare “Stripe only”).
- **undrepost IdP + auctions** — Directus IdP; auctions Wave-2 (not init MUST).

## Hub MoR — what it depends on

Choosing **Hub MoR** (customer pays Undrlla) vs **marketplace/agent** (customer pays each shop / Connect direct charge) depends on:

| Factor | MoR (Undrlla sells) | Non-MoR marketplace |
|--------|---------------------|---------------------|
| **Company / residency** | Where Undrlla legal entity is registered | Same, but lighter “seller” footprint |
| **Where buyers live** | VAT/sales-tax nexus in buyer countries | Often each merchant’s problem |
| **Payment setup** | Stripe as platform selling; or Connect with MoR products | Connect Marketplace / destination charges |
| **Who handles refunds/chargebacks** | **You** | Often merchant |
| **Consumer law** | Distance selling, cooling-off as **seller** | Agent/platform duties differ |
| **Licenses** | Sometimes payment institution / EMI if holding funds | Still KYC on vendors |
| **Tax filings** | You invoice buyers; more filings | Merchants invoice |
| **UX** | Unified cart easy | Unified cart harder |
| **Risk** | High (you’re the store) | Lower, more complex split |

**Practical gate:** until counsel + Stripe/tax setup for MoR in your home market is clear, hub can ship **catalog+search** first and delay charge-as-MoR. Spec currently **targets MoR** for full unified checkout; can revisit without killing catalog.
