# Citizen pricing — discount table

**Status:** aligned with `008-pricing-and-affiliate.md` (2026-07-29)  
**Date:** 2026-07-27 · **Updated:** 2026-07-29  
**Constitution:** §IV.2 — citizens get fee discounts; exact % live here until pricing ops owns it  
**Platform SaaS:** Starter **$29/mo** (migrant/public list). Alpha cohort rules in `008` supersede list price for first 50.

## Principles

1. Discount is a **citizenship benefit**, not a coupon war.  
2. Housing **donation** fee (5% pool) — **do not** discount for citizens in v0 (charity-adjacent optics; everyone pays same into pot).  
3. Stack: citizen rate applies to **platform fee** / list price on commercial products only.  
4. **Direct shop GMV** remains **0.0%** for everyone (`008`); citizen discounts apply to **SaaS / unet / logistics / hub-escrow** where a fee exists.  
5. Numbers below are **starting proposals** — founder may rebalance after first revenue.

## Proposed table (v0)

| Product / fee | Baseline (migrant / public) | Citizen (`undrlla.citizen`) | Notes |
|---------------|----------------------------|----------------------------|--------|
| **undrlla Starter SaaS** (shop host) | **$29/mo** (`008`) | **−10%** ($26.10/mo) | Infra margin thinner than unet |
| **Scale SaaS** | **$89/mo** | **−10%** | Same relative benefit |
| **unet** subscription | list price (TBD) | **−15%** | Sticky benefit |
| **Marketplace platform fee** (shop orders) | **0.0%** direct GMV | **0.0%** (no change) | Zero GMV tax is universal |
| **Hub in-escrow checkout fee** | **3.0–3.5%** (`008`) | **−0.5 pp** | Small citizen perk on hub MoR path only |
| **Logistics job fee** | 12% default | **−1 pp** (12% → 11%) | Small; workers still viable |
| **Housing pool donation fee** | 5% | **0% discount** | Same for all donors |
| **Concierge / Service booking** platform cut | ordinary marketplace fee | same as citizen marketplace treatment | |
| **undrecruiting** placement fee (if any) | TBD | same citizen treatment | No exclusive access |
| **Handle premium** (future) | paid | **−20%** vanity only | Not required for free handle |
| **Turnkey setup Rebate** | $150 one-time, 100% credit | same | Citizenship does not change Rebate |

## Qualification

- Active `undrlla.citizen` status (12 mo path + KYC-lite + clean standing).  
- On `lapsed` citizenship → full price after grace (**recommend end of current billing period**).

## Implementation hooks

- Paddle price_id / subscription map `citizen` vs `default` (pre-LLC).  
- Stripe Coupons / Prices post-LLC.  
- Directus/IdP field `platform_tier` + billing service reads it.  
- Audit: never apply citizen rate to housing pool donation fee path.

## Open

- Exact unet list price.  
- Whether Alpha ($0/6mo) converts to citizen-eligible billing months (recommend **yes** if paid equivalent later).
