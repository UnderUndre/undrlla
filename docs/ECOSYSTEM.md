# Undrlla ecosystem — locked decisions (2026-07-23)

Cross-repo product glue: `undesign`, `undevops`, `undrllanding`, `undrlla`, `undrepost`, `unet`, **`undreseller`** (Medusa marketplace template).

**Product constitution (persons, citizenship, handles, red lines):** [`CONSTITUTION.md`](./CONSTITUTION.md) — v0 locked 2026-07-27. On values/red lines, constitution wins; on pipe detail, feature specs win.

## Marketplace template split (Q8) — 2026-07-27

**Locked:** clean multi-vendor **marketplace template** = [`undreseller`](../../undreseller) (**Medusa monorepo fork**, branch `develop`, `@medusajs/medusa` present). Client shops must not carry undrlla flagship politics (housing pool, citizen, papers).

| Piece | Owner |
|-------|--------|
| Shop template engine | **undreseller** (Medusa) — see `undreseller/UNDERUNRE.md` |
| Flagship digital state + housing + IdP | **undrlla** |
| Deploy client instances | **undevops** (Medusa provision manifest next) |
| Storefront for client shops | **undrllanding** → Medusa Store API |
| Flagship storefront extras | **undrllanding** flags → undrlla / housing |

**Kill dual path:** do **not** build a second full client marketplace on Directus `001` Tier-A. `001` commerce is **legacy / flagship-adjacent** until reduced to polity+housing APIs or formally archived. Hub indexes undreseller shops via **adapter**, not by re-implementing Medusa in Directus.

**Ops note (Windows):** Medusa `www/` docs paths can exceed MAX_PATH — enable `git config core.longpaths true`. Engine under `packages/` is what matters for the template.

## Commerce orchestration (Q9) — **locked 2026-07-27**

**Intent (founder):** undrlla **calls undreseller (Medusa) APIs** so products & services live on Medusa foundation; undrlla does **not** re-implement catalog/cart/checkout in Directus for shops.

### Defaults locked (founder OK)

| # | Decision |
|---|----------|
| Housing pool money | **undrlla-owned** fiat via **Paddle** first (then Stripe post-LLC); optional crypto later via undrepay — not Medusa SoT for pot |
| Bookings / concierge | **SKU on Medusa**; complex housing workflow stays **undrlla** |
| Flagship Medusa | **T2** — dedicated provisioned instance (`TENANT_MODE=flagship`) |
| Directus | Polity + housing runtime **for now** (slim API later optional) |
| Client shop commerce | **Medusa only** — no dual Directus shop |

### Split of brains

| Brain | Owns (source of truth) | Does **not** own |
|-------|------------------------|------------------|
| **undreseller / Medusa** | Products, variants, inventory, carts, orders, payments rails, vendors/sellers, fulfillment, Medusa Admin | Housing pool math, citizen tiers, constitution, papers track |
| **undrlla flagship** | IdP + handles + citizen/migrant; housing applications/pool/escrow; polity APIs; **orchestration** (when to create/update Medusa entities); hub index later | Full e-comm engine for client shops |
| **undrllanding** | UI | Business rules |
| **undevops** | Run instances of Medusa (+ optional undrlla flagship stack) | Product catalog |

### Flagship Undrlla.com = what?

Not “another Medusa storefront only” and not “Directus shop forever”. **Composite:**

```
Browser → undrllanding (flagship flags)
              │
              ├─ auth / @handle / citizen  → undrlla (IdP, Directus or slim API)
              ├─ housing / papers / queue  → undrlla 003 domain
              ├─ shop catalog / cart / pay → undreseller Medusa (flagship instance)
              ├─ unet subscribe            → undrlla billing → may create Medusa product line items OR separate billing
              └─ hub search (later)        → undrlla index fed by Medusa webhooks/sync
```

**Flagship commerce SKUs on Medusa examples:** unet plans, concierge services, undrecruiting packs, merch, donations-as-products (if desired).  
**Flagship domain NOT on Medusa:** `HousingApplication` queue/escrow rules (keep in undrlla; payment capture may still hit Stripe via Medusa *or* undrlla — open).

### How undrlla “дергает” Medusa

| Pattern | Meaning | MVP fit |
|---------|---------|---------|
| **A. Server-side Admin API** | undrlla backend uses Medusa Admin API key / JWT to create products, services-as-products, price lists | Best for “platform creates SKUs” |
| **B. Store API for users** | Browser or BFF uses Medusa Store API for cart/checkout (customer token linked to undrlla user) | Best for shop UX |
| **C. Webhooks Medusa → undrlla** | order.placed, payment.captured → undrlla ledger / housing credit / citizen billing months | Required for polity glue |
| **D. Dual-write UI** | Staff edits both Directus and Medusa | **Reject** — clog |

**Recommended combo:** **A + B + C**. undrlla is **orchestrator + polity**, Medusa is **commerce SoT**.

### Topology (still choose)

| Option | Layout | Pros | Cons |
|--------|--------|------|------|
| **T1 Shared flagship Medusa** | One Medusa for Undrlla.com + N isolated Medusa for clients | Simple flagship | Flagship load shared |
| **T2 All multi-instance** | Every tenant including flagship = own Medusa (undevops) | Hard isolation | Ops weight |
| **T3 Medusa multi-tenancy module** | One cluster, tenant_id in Medusa | Fewer DBs | Weaker isolation, Medusa MT maturity |

**Lean T2 for clients + T1-or-T2 for flagship** (flagship can be “just another provisioned Medusa” with special env `TENANT_MODE=flagship`).

### What happens to undrlla `001` Directus commerce

| Was in `001` | After Q9 |
|--------------|----------|
| Product, Cart, Order, Booking as Directus SoT for **client shops** | **Deprecated for clients** → Medusa |
| Same for **flagship shop SKUs** | **Move to flagship Medusa** over time |
| Payment abstraction in Directus | Prefer Medusa payment modules; undrlla listens webhooks |
| Extension points for housing | **Stay undrlla**; housing may *reference* Medusa `order_id` / `payment_id` |
| Provisioning portal `002` | Manifest targets **Medusa app** images, not only Directus shop |

### Build order (revised draft)

```
1. undreseller: runnable template-shop (Medusa app) + smoke
2. undevops: provision Medusa instance (client)
3. undrllanding Wave-1: client shop → Medusa Store API
4. undrlla: IdP + handles (+ citizen later)
5. undrlla ↔ Medusa glue: map undrlla user ↔ Medusa customer; webhooks
6. flagship Medusa: undrlla-created SKUs (unet, concierge) via Admin API
7. housing 003 on undrlla (pool/escrow); optional Medusa product for “donation sub”
8. hub: index Medusa catalogs (not Directus products)
```

### Payments (Q10) — **locked 2026-07-27, amended Paddle-first** · see `undrepay` + `Paddle-into-JS-TS-web-app.md`

| Rail | Now (pre–US LLC) | Later (after US LLC) | Local / CI test |
|------|------------------|----------------------|-----------------|
| **Fiat cards / subs / many digital sales** | **Paddle Billing (MoR)** — `@paddle/paddle-js` + `@paddle/paddle-node-sdk` | **Stripe** (Connect/marketplace where needed) | Paddle **sandbox** (`test_` client token + sandbox API key); later Stripe test mode |
| **Crypto + internal ledger / holds** | **undrepay** (Blnk + SHKeeper) | same | undrepay compose + SHKeeper fixtures |
| **Fake full Stripe API proxy** | **Out of scope** | still out | — |

**Why Paddle first:** founder operates without US LLC; Paddle is **Merchant of Record** (tax/VAT/PCI burden on Paddle). Reference guide: repo root `Paddle-into-JS-TS-web-app.md`.

**Why Stripe later:** multi-vendor marketplace Connect, broader US banking, logistics hold/capture patterns — after LLC.

**Paddle constraint (honest):** Paddle MoR fits **single-seller / SaaS / platform-owned catalog** (flagship SKUs, housing pool sub, unet, shop license fee, simple one-vendor template shop). It is a **poor fit for classic multi-vendor Connect** (many independent sellers, platform only takes fee). Template-shop MVP = **single seller (or platform MoR)** on Paddle; multi-vendor Connect = **post-Stripe phase**.

**undrepay:** crypto + Blnk only — does **not** replace Paddle/Stripe for cards.

**Medusa template-shop:** integrate **Paddle checkout** (custom overlay/inline + webhooks), not Medusa-default Stripe-only path, until LLC.

### Integration contract (Q11) — draft 2026-07-27

Cross-repo payment + webhook contract: [`../../undrepay/specs/INTEGRATION.md`](../../undrepay/specs/INTEGRATION.md).

**Lean locks in that doc:** correlation metadata; undrepay-native intents; shop crypto completes via Medusa then Medusa→undrlla; housing pool **fiat = Paddle first** (Stripe post-LLC); browser never holds undrepay master keys; shared undrepay cluster keyed per Medusa instance.

## Identity (Q1)

**Locked: one login for the ecosystem.**

| Choice | Decision |
|--------|----------|
| Model | **Single identity** — one account across undrlla (shop/housing), undrllanding, undrepost, unet subscriber portal |
| Authority | **Flagship undrlla (Directus users)** is the IdP for MVP |
| Consumers | undrllanding, undrepost (Member mirror / JWT), unet billing UI — validate tokens issued by undrlla |
| Client shops | Medusa customer session **and/or** federated undrlla JWT (see **`005-sso-jwt-contract.md`**) |
| Deferred | Separate auth microservice only if scale/compliance forces it |
| **Platform tiers** | Default **`undrlla.migrant`**; earn **`undrlla.citizen`** after **12 mo** qualifying billing + KYC-lite + clean standing (`CONSTITUTION` §IV). Dual labels: not a state passport. |
| **Handle** | Global unique `@username`; hub `undrlla.com/@user`; TM/country reclaim to lawful holders (`CONSTITUTION` §V). |
| **SSO / JWT (A7)** | **Locked 2026-07-29** in [`005-sso-jwt-contract.md`](./005-sso-jwt-contract.md): Zero-Code Auth — Directus native IdP + `@medusajs/auth`; RS256/JWKS; cookie domain for `*.undrlla.network` (or prod flagship domain); OIDC PKCE for custom merchant domains |
| **Pricing / affiliate (A9)** | **Locked** in [`008-pricing-and-affiliate.md`](./008-pricing-and-affiliate.md): Alpha 50 × 6 mo free; Starter $29/mo; 0% direct GMV; Rebate $150; 20% RevShare USDT; hub fees split |
| **KYC housing (D9)** | **MVP = manual/agency** (admin or contracted compliance agency). No mandatory Sumsub/Onfido. See `003` FR-012. |

**Recommendation rationale:** one brain for “who is this user”; undrepost’s old Better-Auth/Medusa IdP assumption is obsolete — rebind to Directus native auth.

## undrllanding shape (Q2) — plain language

**Problem:** do we build one website app or two?

| Option | Meaning |
|--------|---------|
| **A — One app, many modes** | Same Next.js codebase (`undrllanding`). Env flags turn modules on/off. Clean client shop: only catalog/cart. Flagship Undrlla.com: + housing, hub, VPN, … |
| **B — Two apps** | `storefront` for clients, `flagship-web` for Undrlla.com — separate deploys, more code to maintain |

**Locked: Option A — one codebase, feature flags.**

- undevops deploys the **same image** with different env: `DIRECTUS_URL`, `TENANT_MODE=client|flagship`, `FEATURES=...`.
- Wave 1 UI: shop only (catalog, cart, checkout, booking).
- Later waves: housing queue, hub, unet subscribe — only when `FEATURES` allow.

## undrepost timing (Q3)

**Founder lean: before hub/housing. Recommendation: after shop deploy works, before or parallel early housing — not before undrlla Tier-A.**

**Locked order:**

1. undrlla Tier-A core + undesign  
2. undevops one-click (shared manifest)  
3. undrllanding Wave 1 (shop)  
4. **undrepost Wave 1** (blog + SSO to undrlla IdP) — content/social; **not** blocking housing  
5. undrlla housing pot+queue  
6. unet paid + hub/logistics  

undrepost **before** hub is fine; **not** before a working shop. Housing does **not** depend on undrepost if MVP chat = Telegram.

## unet (Q4)

**Locked: consumer product with money** — not only dev port-forward.

| Piece | Owner |
|-------|--------|
| Tunnel engine | `unet` binary (rootless AmneziaWG + netstack) |
| Sell / subscribe / revoke / device limits | **undrlla** flagship (billing + peer config issuance) |
| Gateway / exit | ops via undevops; control API TBD |
| Flow score / MAQ accelerators | undrlla reads node uptime (later) |

## undevops topology (Q5) — plain language

**Question was:** one computer farm for all shops, or many regions?

| | |
|--|--|
| **Single cluster (MVP)** | One undevops control plane + one main region. All client shops as containers there. Simple, cheap. |
| **Multi-region** | EU shop in EU, US in US — latency, law, cost. Later. |

**Locked: single region / single undevops control plane for MVP.** Multi-region when revenue and latency force it.

## Chat (Q6)

**Locked: Telegram only in MVP** (already primary notifications in undrlla).  
undrepost DMs / in-app chat — post-MVP.

## Provision contract (Q7)

**Locked: undrlla `002` ProvisioningManifest is the source of truth.**  
undevops implements ingestion of that manifest (adapt/replace thin `003-undrlla-one-click-deploy` payload). One schema, one job status API.

---

## Suggested build order

**Superseded for launch sequencing** by [`ECOSYSTEM-REQUIREMENTS.md`](./ECOSYSTEM-REQUIREMENTS.md) §5 (Phase A Medusa shop first). Historical order below is **pre-Medusa-pivot** and must not drive client-commerce work:

```
# LEGACY (do not use for client shops):
undesign → undrlla Directus Tier-A → undevops Directus one-click → undrllanding Directus…

# CURRENT (2026-07-29):
undreseller template-shop + Paddle sandbox
  → undevops Medusa provision
  → undrllanding Medusa Store API
  → undrlla IdP + housing pool fiat (Paddle pre-LLC; Stripe post-LLC)
  → undrepay crypto + glue webhooks
  → undrepost / unet / hub (later)
```

## Open (not locked)

- unet price plans / list SKU  
- Multi-region criteria (when revenue/latency force it)  
- Whether **client-only** shops stay isolated Medusa logins forever or always federate to flagship IdP (Gate 0–1 = Medusa-only OK; multi-app flagship path uses `005`)  
- Hub MoR legal/tax gate (catalog+redirect first; charge-as-MoR after counsel)  
- O1–O4 defaults in `undrepay/specs/INTEGRATION.md` (founder ACK)

### Closed (do not re-open without constitution/ECOSYSTEM change)

| Item | Locked in |
|------|-----------|
| Cookie domain / JWT / SSO | [`005-sso-jwt-contract.md`](./005-sso-jwt-contract.md) — RS256/JWKS, `.undrlla.network` cookie, OIDC PKCE for custom domains |
| Fiat rails | Q10 — **Paddle MoR pre–US LLC**; **Stripe post-LLC** |
| Housing pool fiat SoT | undrlla (not Medusa); rail = Paddle first, Stripe later |
| Client shop commerce | Medusa only (`undreseller`) — no Directus dual path |
| Pricing / affiliate SaaS | [`008-pricing-and-affiliate.md`](./008-pricing-and-affiliate.md) |

## SpecKit review fixes applied (2026-07-23)

| Item | Status |
|------|--------|
| undevops 003 ingests undrlla ProvisioningManifest | Done |
| undrllanding Wave-1; TG auth Wave-2 | Done |
| undrepost IdP = Directus; auction deferred | Done |
| undrlla FR-040 roles, FR-041 delivery, FR-042 hub phase, FR-043 unet | Done |
| 003 HousingPoolSubscription + chargeback + FR-024 fee | Done |
| undevops 002 plan/tasks + group ACL | Done |
