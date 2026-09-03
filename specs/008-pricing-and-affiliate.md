# 008 — Pricing, Economics & Affiliate Engine (Specification)

**Status**: Active / Approved  
**Date**: 2026-07-28  
**Depends on**: `ECOSYSTEM.md`, `005-sso-jwt-contract.md`, `002-provisioning-portal/spec.md`, `undrepay/specs/DECISIONS.md`  

---

## 1. Intent & Monetization Philosophy

Define the business model, pricing tiers, setup-fee rebate program, central hub transaction rules, and crypto-native affiliate engine for the Undrlla ecosystem.

> ### Core Economic Laws
> 1. **Zero GMV Tax on Standalone Sales**: We charge **0.0% transaction fee** when customers purchase on the merchant's standalone site (`myshop.com`).
> 2. **Central Hub Streaming (`list_on_hub: true` by default)**: Client catalog items stream automatically to `undrlla.network` for free organic traffic. Direct redirects = 0.0% fee; flagship in-hub escrow checkouts = 3%–5% platform fee.
> 3. **Dual-Currency Display**: Products on Central Hub render both native merchant currency + auto-converted user locale currency (e.g. `1,500 ₺ (~$45.00 USD)`).
> 4. **Rebate Setup Model**: Upfront $150 custom setup fees are **100% credited back** as monthly hosting discounts ($25/mo off for 6 months). Onboarding is conducted via 1-on-1 human Telegram dialogue.
> 5. **Crypto-Native Affiliates**: External promoters earn **20% perpetual recurring RevShare** with direct USDT/crypto payouts via **SHKeeper**.

---

## 2. Pricing Architecture (2026)

| Tier | Price | Included Scope | Standalone GMV Fee | Hub Direct Fee | Hub Escrow Fee |
|------|-------|----------------|-------------------|----------------|----------------|
| **Alpha Cohort (First 50)** | **$0 / mo** | 1 Marketplace, Medusa 2.0, Paddle/Stripe + SHKeeper, custom domain, white-glove Telegram setup | **0.0%** | **0.0%** | **3.0%** |
| **Starter (Lite)** | **$29 / mo** | 1 Marketplace, Medusa 2.0, custom Next.js 15 storefront, automated SSL, Telegram Mini App | **0.0%** | **0.0%** | **3.5%** |
| **Scale (Pro)** | **$89 / mo** | Isolated Postgres + Redis container, dedicated CPU/RAM, multi-vendor support, SLA 99.9% | **0.0%** | **0.0%** | **3.0%** |
| **Enterprise** | **$299+ / mo** | Multi-region deployment, custom SHKeeper node, dedicated SLA & engineer | **0.0%** | **0.0%** | **2.5%** |

---

## 3. Turnkey Setup & 1-on-1 Telegram Onboarding (FDE Rebate)

For clients requiring custom design, theme development, or legacy migrations:
- **Base Rate**: **$25 – $35 / hour**.
- **Turnkey Setup Package**: **$150** (6 hours of custom engineering & design adaptation).
- **Human Onboarding Workflow**: No cold robotic forms. A dedicated designer/engineer initiates a 1-on-1 Telegram dialogue with the client to collect brand assets (logo, font preference, color references, product seeds).
- **100% Rebate Mechanism**: The $150 setup fee is converted into monthly infrastructure credits ($25/mo discount applied to the client's $29/mo sub for 6 months).
- **Outcome**: Custom onboarding is net-free for the client over time, while Undrlla secures upfront cashflow and a long-term subscriber.

---

## 4. Central Hub Traffic & Currency Rules (`list_on_hub`)

- **Default Opt-In**: Newly provisioned marketplaces default to `list_on_hub: true` and `share_catalog_to_hub: true` so merchants receive instant organic buyers from day one. Merchants can toggle this off anytime in 1 click.
- **Dual-Currency Formatting**: Hub search renders native merchant price alongside live auto-converted user currency (`TRY`, `EUR`, `USD`, `USDT`).
- **Fee Split**:
  - Redirect link to merchant's domain (`myshop.com`) -> **0.0% fee**.
  - Direct flagship checkout & escrow on `undrlla.network` -> **3.5% fee**.

---

## 5. Affiliate & Referral Engine (20% Perpetual RevShare)

### 5.1 Internal Client Referral Track (Discount Loop)
- **1 Active Referral**: 50% discount on monthly subscription/support ($14.50/mo savings).
- **2 Active Referrals**: 100% free monthly hosting and maintenance ($0/mo sub).
- **3+ Active Referrals**: Credits accumulate into internal balance for custom dev hours ($25/hr value).

### 5.2 External Promoter / Affiliate Track (Crypto Cashout)
- **Commission Rate**: **20% Perpetual Recurring RevShare** on all active subscription renewals ($5.80/mo per $29/mo plan).
- **Payout Rails**: Non-custodial crypto payouts via **SHKeeper** (USDT TRC20 / BEP20 / TON / BTC).
- **Minimum Payout Threshold**: **$50 USDT**.
- **Holding Period (Anti-Fraud)**: 30-day locking period (`pending → available`) to buffer for chargebacks and refunds.

---

*End of Specification.*
