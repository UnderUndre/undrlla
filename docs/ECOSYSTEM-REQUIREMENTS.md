# Undrlla ecosystem — requirements map, readiness & launch plan

**Date:** 2026-07-27  
**Status:** living audit (product law)  
**Audience:** founder + agents  
**Verdict (honest):** **нет, «всё от и до» не готово.** Политика и housing-логика проработаны глубже, чем runnable shops/payments. После pivot **Medusa (undreseller)** часть старых `001`/`undrllanding`/`002` спек **устарела по backend** — решения в `ECOSYSTEM` Q8–Q11 побеждают, но **не переписаны** во все feature-specs.

**Sources of truth (priority):**

1. [`CONSTITUTION.md`](./CONSTITUTION.md) — values, citizen, red lines  
2. [`ECOSYSTEM.md`](./ECOSYSTEM.md) — cross-repo locks (Q1–Q11)  
3. Feature specs when they **don’t conflict** with (1)(2); on conflict → fix the feature spec  
4. [`../undrepay/specs/DECISIONS.md`](../../undrepay/specs/DECISIONS.md) + [`INTEGRATION.md`](../../undrepay/specs/INTEGRATION.md)  
5. [`../undreseller/UNDERUNRE.md`](../../undreseller/UNDERUNRE.md)  
6. This file — readiness + launch sequencing  

---

## 0. One-screen architecture (target)

```
                    ┌─────────────────────────────────────┐
                    │     undrllanding (Next, flags)      │
                    │  client shop | flagship Undrlla.com │
                    └───────────┬─────────────┬───────────┘
                                │             │
              Store API / JWT   │             │ polity / housing APIs
                                ▼             ▼
              ┌─────────────────────┐   ┌──────────────────┐
              │ undreseller Medusa  │   │ undrlla flagship │
              │ (per-tenant inst.)  │   │ IdP · housing ·  │
              │ catalog cart order  │   │ citizen · glue   │
              └─────────┬───────────┘   └────────┬─────────┘
                        │                        │
           ┌────────────┼────────────┐           │
           ▼            ▼            ▼           ▼
        Paddle      undrepay      Admin API   Paddle (pool)
        (cards now) (crypto+ledger) (SKU create) (housing fiat)
        Stripe later                     Stripe later (post-LLC)
                        │
                   Blnk + SHKeeper

 undevops ── provision Medusa (+ later flagship stack)
 undrepost ─ content/SSO (after shop)
 unet ────── tunnel binary; sell via undrlla (+ Medusa SKU later)
 undesign ── tokens/theme
```

---

## 1. Readiness by repo

| Repo | Role | Spec maturity | Code / product readiness | Notes |
|------|------|---------------|--------------------------|--------|
| **undesign** | Design tokens | Low (pkg) | ✅ usable package | Exists on npm path |
| **undreseller** | Marketplace engine (Medusa) | Outline only (`UNDERUNRE`) | 🟡 monorepo present; **no** consumer template-shop app locked | Windows `www/` path issues; engine `packages/` OK |
| **undrlla** | Polity + housing + IdP + orchestration | 🟡 deep on Directus `001`–`003`; **pivot incomplete** | 🟡 Directus fork base; commerce SoT **must not** stay client path | `CONSTITUTION` v0; outlines 004/006/007 |
| **undrepay** | Crypto + ledger | 🟡 decisions + integration + **tasks.md** | 🔴 no app yet | Stripe-proxy doc DEPRECATED |
| **undevops** | Deploy | 🟡 Dokploy fork; **003 Medusa plan/tasks** | 🟡 platform exists; job **code** TBD | Manifest schema SoT in undrlla `002` contracts |
| **undrllanding** | UI | ✅ storefront Wave-1 Medusa-targeted | 🟡 landing + waitlist live; shop API later | Spec retargeted 2026-07-28 |
| **undrepost** | Content/social | 🟡 Payload init specs | 🟡 fork base | IdP → undrlla; after shop |
| **unet** | VPN engine | 🟡 netstack tunnel specs | 🟡 Go engine | Sell/subscribe undrlla; gateway TBD |

**Legend:** ✅ ready enough to use · 🟡 partial · 🔴 missing  

---

## 2. Requirements catalog (end-to-end)

Grouped by **launch slice**, not by repo. Each requirement: **ID · statement · status · owner**.

### Status key

| Tag | Meaning |
|-----|---------|
| **L** | Locked decision (ECOSYSTEM / CONSTITUTION / 003) |
| **S** | Spec exists (may be legacy backend) |
| **O** | Outline only |
| **G** | Gap — need decide or write |
| **X** | Explicitly deferred (post-MVP / later phase) |

---

### A. Identity & digital polity

| ID | Requirement | Status | Owner |
|----|-------------|--------|--------|
| A1 | Single ecosystem identity (one login) | **L** | undrlla IdP |
| A2 | Flagship undrlla issues tokens; consumers validate | **L** | undrlla → landing/repost/unet |
| A3 | Dual labels: `undrlla.migrant` vs `undrlla.citizen` ≠ state passport | **L** | CONSTITUTION |
| A4 | Citizen earn: 12 mo qualifying billing + KYC-lite + clean ToS | **L** | undrlla |
| A5 | Citizen benefits: fee discounts, voice, badge — **no** housing queue boost | **L** | undrlla + pricing |
| A6 | Global unique `@username` + hub page + TM/country reclaim | **L** / **O** | `004-identity-handles` |
| A7 | Cookie domain + JWT format cross-app | **L** | `005-sso-jwt-contract` |
| A8 | Client shops: local auth vs flagship SSO | **L** | `005-sso-jwt-contract` (Directus Native + `@medusajs/auth`) |
| A9 | Citizen discount & pricing economics locked | **L** | `008-pricing-and-affiliate` |
| A10 | Constitution amend / poll process operationalized | **O** | CONSTITUTION §X–XII |

---

### B. Client marketplace (sell products/services)

| ID | Requirement | Status | Owner |
|----|-------------|--------|--------|
| B1 | Commerce engine = **Medusa** (undreseller), not Directus, for client shops | **L** | Q8–Q9 |
| B2 | Runnable template-shop (deployable Medusa app) | **S** SpecKit `001-template-shop` ready; **code G** | undreseller |
| B3 | Multi-vendor catalog, cart, checkout | **S** single-seller MVP; multi-vendor post-Stripe | undreseller |
| B4 | Fiat via **Paddle** first (sandbox/live); **Stripe** after US LLC | **L** | undreseller + Paddle → later Stripe |
| B5 | Crypto checkout via undrepay provider | **L** / **S** contract + tasks | undrepay + Medusa plugin |
| B6 | Vendor invite + Connect/crypto payouts | **S** intent; post-Stripe | undreseller |
| B7 | Domain BYO + platform subdomain + TLS | **S** `002` domain_mode + `008` | undevops |
| B8 | undevops one-click provision **Medusa** instance | **S** 003 Medusa plan/tasks; **code G** | undevops |
| B9 | Storefront Wave-1: catalog/cart/checkout on Medusa API | **S** undrllanding 001 retargeted; **code G** | undrllanding |
| B10 | Branding / undesign tokens | **L** intent | undesign + landing |
| B11 | TG Mini App shop | **X** Wave-2 | undrllanding |
| B12 | SaaS billing for shop instance (who pays rent) | **G** | business + undrlla |
| B13 | Notifications (email/TG) for orders | **S** intent | landing + Medusa |

---

### C. Flagship Undrlla.com (composite)

| ID | Requirement | Status | Owner |
|----|-------------|--------|--------|
| C1 | Flagship = IdP + housing + shop SKUs on **dedicated Medusa** (T2) | **L** | undrlla + undreseller |
| C2 | undrlla Admin API orchestration creates/updates Medusa SKUs | **L** intent | undrlla |
| C3 | Storefront flagship flags: housing, hub, unet modules | **L** | undrllanding |
| C4 | User map undrlla_user ↔ medusa_customer | **G** / contract O | glue |
| C5 | Webhooks Medusa → undrlla for order.paid / citizen months | **L** / **O** | INTEGRATION |
| C6 | unet plans sold (billing + peer config) | **L** intent; gateway **G** | undrlla + unet + undevops |
| C7 | Founder concierge services as bookable SKUs | **S** `003` + Medusa SKU | undrlla + Medusa |

---

### D. Housing + path-to-papers

| ID | Requirement | Status | Owner |
|----|-------------|--------|--------|
| D1 | Shared pot + queue model B; ≥$1/mo keeps seat | **L** **S** `003` | undrlla |
| D2 | Default need $500k; self-serve ≤500k; over-cap admin | **L** **S** | undrlla |
| D3 | 5% donation fee; no self-serve refund | **L** **S** | undrlla |
| D4 | Escrow to notary only; never cash to applicant | **L** **S** | undrlla |
| D5 | Recommended Tier-1 expansion housing: **Cyprus (CY €300k Fast-Track PR)**, **UAE (AE 2M AED Golden Visa)**, **Panama (PA $300k QIV)**, **Greece (GR €250k conversion)**, **Malta (MT €375k MPRP)**; Primary Home: **Canada Alberta (CA-AB)**, **US (US-NC / US-WA)**; TR (SPK gap/seismic warning), GE (1% IT tax setup, Batumi mold warning), US-TX (2.5% prop tax trap), US-FL (insurance crisis / SB 264 law); ES/PT blacklisted (Golden Visa abolished 2025/2023); admin ServiceArea | **L** **S** | undrlla |
| D6 | Housing country ⟂ papers track country | **L** **S** FR-025 | undrlla |
| D7 | Housing pool money via **undrlla** (not Medusa SoT): **Paddle MoR pre–US LLC**, **Stripe post-LLC** | **L** | undrlla |
| D8 | Optional housing crypto donate via undrepay | **O** later | undrepay + undrlla |
| D9 | KYC/AML for housing applicants & escrow destinations | **L** MVP = **manual/agency** (no mandatory Sumsub/Onfido); post-MVP optional provider | `003` FR-012 · ops + undrlla |
| D10 | Multi-vendor housing pros | **X** post-MVP FR-019 | undrlla |
| D11 | Rental PM / smart home | **X** post-003 | — |
| D12 | Legal entity + compliance for large pools | **G** counsel | founder |

---

### E. Payments (cross-cutting)

| ID | Requirement | Status | Owner |
|----|-------------|--------|--------|
| E1 | Fiat: Paddle now; Stripe post-LLC | **L** | Paddle + later Stripe |
| E2 | Crypto + ledger: undrepay (Blnk+SHKeeper) | **L** | undrepay |
| E3 | No full fake-Stripe proxy MVP | **L** | DECISIONS |
| E4 | Integration contract (metadata, webhooks, paths) | **O** draft | INTEGRATION.md |
| E5 | undrepay implement tracer + compose | **G** | undrepay |
| E6 | Medusa undrepay payment provider | **G** | undreseller plugin |
| E7 | Logistics hold/capture on accept/complete | **L** intent | later `005` + undrepay ledger_hold |

---

### F. Hub, logistics, social, match, recruit

| ID | Requirement | Status | Owner |
|----|-------------|--------|--------|
| F1 | Hub federated catalog + unified cart + MoR | **L** intent **X** build | future `004-hub` |
| F2 | Hub legal/tax MoR gate | **G** counsel | founder |
| F3 | Logistics workforce 12% hold/capture | **L** intent **X** | `005` |
| F4 | undrepost Wave-1 blog + SSO | **S** after shop | undrepost |
| F5 | Chat = Telegram MVP | **L** | ops |
| F6 | Union matching (serious dating) | **O** **X** early | `006` |
| F7 | undrecruiting IT + CBI crew | **O** **X** | `007` |
| F8 | MAQ / barter / AI builder | **X** | `001` deferred |

---

### G. Ops, legal, launch hygiene

| ID | Requirement | Status | Owner |
|----|-------------|--------|--------|
| G1 | Single-region undevops MVP | **L** | undevops |
| G2 | Secrets, backups, monitoring | **G** runbook | undevops |
| G3 | Legal entity, ToS, privacy, KYC vendor | **G** | founder + counsel |
| G4 | Trademark/handle policy ops | **O** | undrlla |
| G5 | No passport guarantee / no sham marriage in copy | **L** | CONSTITUTION |
| G6 | Production domains, DNS, TLS | **G** | undevops |
| G7 | Support channel (TG) | **L** intent | founder |

---

## 3. Critical gaps & contradictions

| # | Issue | Risk | Resolution |
|---|--------|------|------------|
| 1 | **Directus `001` vs Medusa Q8/Q9** | Dual build / wasted work | Archive client-commerce FR from `001` as historical; new Medusa template FR / UNDERUNRE + SpecKit later |
| 2 | **Build order in ECOSYSTEM** still says “undrlla Tier-A first” | Wrong sequence | Use §5 launch plan below |
| 3 | **002 ProvisioningManifest** still Directus-shaped | Can’t one-click Medusa | Write Medusa manifest schema; undevops ingest |
| 4 | **undrllanding** API contract → Directus | Shop won’t talk Medusa | Retarget Wave-1 to Store API |
| 5 | **undrepay** not built | Crypto path blocked | Phase C after Stripe shop works |
| 6 | **SSO cookie/JWT** | Was open | **Closed** — `005-sso-jwt-contract.md` (Zero-Code Directus + Medusa auth) |
| 7 | **KYC provider** | Was open for automation | **Closed for MVP** — manual/agency per `003` FR-012; Sumsub/Onfido optional later |
| 8 | **Hub MoR legal** | Early hub charge = liability | Catalog-only hub until counsel |
| 9 | **Shop instance SaaS fee** | No revenue model for undevops cost | Decide monthly fee or founder absorb |
| 10 | **FLOWS-AUDIT** pre-Medusa | Stale gaps list | Re-audit after Phase A |

---

## 4. What “MVP launch” means (three gates)

Do **not** launch “whole digital state” day one. Three gates:

### Gate 0 — Internal dogfood (no public money)

- Medusa template-shop up locally  
- **Paddle sandbox** checkout  

- undevops can deploy one shop (even manual runbook)  
- undrllanding client mode browses that shop  

### Gate 1 — Public **client shop** (first revenue shape)

- Real domain + TLS  
- **Paddle live** (or Stripe after LLC)  

- One merchant + products  
- Basic support TG  
- **No** housing, hub MoR, citizen required  

### Gate 2 — Flagship **Undrlla.com** thin

- IdP + register/login  
- Housing pool subscribe (Stripe) + apply (CY/AE/PA priority; TR/GE advisory notes) + founder concierge  
- Optional: flagship Medusa for merch/unet SKU  
- Copy: no passport guarantee  

### Gate 3 — Crypto + polity polish

- undrepay crypto on shop  
- `@handle` hub  
- citizen clock starts counting  
- hub catalog (no MoR charge until legal)  

---

## 5. Action plan (recommended sequence)

### Phase A — Commerce foundation (2–6 weeks depending on focus)

| # | Action | Repo | Done when |
|---|--------|------|-----------|
| A1 | Create **runnable** Medusa app (`apps/template-store` or sibling) | undreseller | `yarn dev` + admin + store API |
| A2 | Paddle sandbox payment end-to-end | undreseller template-shop | sandbox pay → order paid (see TEMPLATE-SHOP-PLAN) |
| A3 | Document env vars + seed script | undreseller | README path |
| A4 | Draft **Medusa ProvisioningManifest** | undrlla `002` or undevops | schema + example JSON |
| A5 | undevops job: deploy template-store image | undevops | one click or scripted deploy |
| A6 | undrllanding Wave-1 **Medusa** catalog/cart/checkout | undrllanding | talks to template Store API |
| A7 | Mark `001` client-commerce as **legacy / do not implement for clients** | undrlla specs | note in 001 + ROADMAP |

### Phase B — Flagship identity + housing money (parallel after A1)

| # | Action | Repo | Done when |
|---|--------|------|-----------|
| B1 | IdP: register/login JWT + refresh cookie plan | undrlla | doc + minimal API |
| B2 | A7 lock: cookie domain | undrlla | written in ECOSYSTEM |
| B3 | Housing pool **Paddle** (sandbox/live) + subscription ≥$1; Stripe after LLC | undrlla | payment → pool ledger |
| B4 | Housing application + queue model B (no multi-country UI polish) | undrlla | apply → verified path (manual KYC OK) |
| B5 | Admin: ServiceArea TR/GE, allocate, escrow **manual** confirm | undrlla | ops can complete one dry-run |
| B6 | Flagship undrllanding: housing screens | undrllanding | FEATURES=housing |

### Phase C — Glue + crypto

| # | Action | Repo | Done when |
|---|--------|------|-----------|
| C1 | Freeze INTEGRATION O1–O4 | undrepay | founder ACK |
| C2 | undrepay tracer (Blnk create/capture) + compose | undrepay | integration tests green |
| C3 | SHKeeper path + HMAC | undrepay | crypto fixture paid |
| C4 | Medusa undrepay provider | undreseller | crypto next_action UI |
| C5 | undrlla_user ↔ medusa_customer + order.paid webhook | undrlla | citizen month can tick from shop |

### Phase D — Expand (only after Gate 1 or 2)

| # | Action |
|---|--------|
| D1 | Handles public `@user` |
| D2 | unet paid + gateway |
| D3 | undrepost SSO + blog |
| D4 | Hub catalog index (no MoR) |
| D5 | Logistics `005` |
| D6 | undrecruiting / union outlines → specs when capacity |

---

## 6. Requirements completeness checklist

### Product decisions — mostly enough for **Gate 0–1**?

| Area | Enough to build? |
|------|------------------|
| Who owns shop commerce | **Yes** (Medusa) |
| Who owns housing | **Yes** (undrlla 003) |
| Who owns cards vs crypto | **Yes** (Stripe / undrepay) |
| Citizen rules | **Yes** for later gate |
| SSO details | **Yes** — `005-sso-jwt-contract` (implement when multi-app) |
| Medusa provision | **Schema yes** (`002` Medusa manifest) — undevops job still A5 |
| KYC vendor | **MVP yes (manual/agency)** — automation optional |
| Hub MoR | **No** — defer |
| Legal entity | **No** for live marketplace |

### Spec completeness — “от и до”?

| | |
|--|--|
| Constitution / polity | **Good enough v0** |
| Housing domain | **Strong** (`003` + papers ⟂) |
| Cross-repo glue | **Good** ECOSYSTEM + INTEGRATION draft (hygiene 2026-07-30: Paddle/SSO aligned) |
| Medusa shop FR | **Medium** — undreseller `001-template-shop` SpecKit (code still G) |
| undrepay code specs | **Medium** — decisions + integration + tasks.md; no reviews |
| undevops Medusa | **Medium** — 003 retargeted; job code G |
| undrllanding Medusa | **Medium** — retargeted Wave-1 (code lag) |
| Full ecosystem E2E tests plan | **Missing** |

**Conclusion:** requirements are **directionally complete for a phased launch**, **not** complete as one big-bang “digital state day 1”. Biggest hole is **runnable commerce path (Medusa + provision + storefront)** after the pivot.

---

## 7. Suggested “definition of ready” before writing more features

Before promoting union/recruiting/hub to full SpecKit:

1. [ ] Phase A Gate 0 green  
2. [ ] ECOSYSTEM open items: JWT/cookie  
3. [ ] `001` legacy banner for client commerce  
4. [ ] Medusa ProvisioningManifest v0  
5. [ ] INTEGRATION O1–O4 ACK  

---

## 8. Immediate next discussion / work (pick one)

| Code | Focus |
|------|--------|
| **A** | Start Phase A execution (template-shop) |
| **S** | Rewrite stale specs (`001` banner, undrllanding Medusa FR, Medusa manifest) **before** code |
| **L** | Legal/ops checklist (entity, KYC vendor, Stripe account, domains) only |
| **B** | Phase B housing Stripe dry-run design deeper |

**Recommendation:** **S then A** — half-day “spec hygiene” so agents don’t implement Directus shops, then **template-shop**.

---

## Changelog

| Date | Change |
|------|--------|
| 2026-07-27 | Initial ecosystem requirements + readiness + launch plan (post Medusa/undrepay pivot) |
| 2026-07-29 | Spec hygiene: A7/A8/A9 locked (`005`, `008`); D9 KYC MVP = manual/agency; Medusa ProvisioningManifest v2 schema; `001` legacy banner hardened |
| 2026-07-30 | Hygiene S: D7 housing fiat = Paddle pre-LLC (not bare “Stripe”); ECOSYSTEM Open cleaned (SSO closed); INTEGRATION + undrepost + `002` plan aligned |
