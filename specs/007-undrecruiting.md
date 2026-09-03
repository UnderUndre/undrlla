# 007 — undrecruiting (Dev Guild & Marketplace Workforce Spec)

**Status:** Approved / Spec
**Date:** 2026-07-30 (Updated with Dev Guild Modes A/B & Open-Source Bounty Integration)
**Constitution:** §IX
**Builds on:** `001` Service/Booking/payouts, `undrepay` integration, Opire/Algora OSS bounty architecture

---

## 1. Intent & Core Architecture

`undrecruiting` provides a **proof-over-theater** workforce engine for two key verticals:
1. **IT / Builder Guild** — open-source bounty board, staff-augmentation, and verified skill profiles tied to `@handle`.
2. **Housing & Polity Ops Crew** — verified lawyers, realtors, translators, and escorts.

### Dual Operating Modes for Dev Guild (Modes A & B)

To optimize founder time while scaling technical output:

- **Mode A — Founder-as-Architect (Managed Agency)**:
  - Founder conducts initial scoping & spec writing (`/speckit.specify`).
  - Task assigned to Guild developer.
  - Founder reviews code (`/code_review`) and manages client delivery.
  - **Economics**: 30% platform/architect fee, 70% developer payout via `undrepay`.

- **Mode B — Autonomous P2P Bounties (Marketplace Escrow)**:
  - Activated when founder has zero time for active code review.
  - Clients post task bounties directly to GitHub issues (Opire/Algora pattern).
  - Developers submit PRs; automated test suites + peer review unlock `undrepay` escrow.
  - **Economics**: 5-10% automated platform protocol fee.

---

## 2. Requirements

- **FR-R01**: System MUST support developer and operational crew profile registration linked to ecosystem `@handle`, tracking verified skill badges, language fluencies (`languages[]`), PR commits, trial scores, and completed project count.
- **FR-R02**: System MUST integrate GitHub Issue Bounty webhooks (Opire/Algora pattern) allowing bounty creation and auto-payout via `undrepay` smart intents (`upi_...`).
- **FR-R03**: System MUST allow switching between Mode A (Managed Agency) and Mode B (P2P Bounties) via founder profile toggle.
- **FR-R04**: All crypto payouts for bounties, crew packs, and client projects MUST use `undrepay` (Blnk + SHKeeper).
- **FR-R05**: Verified developer skill badges MUST be publicly viewable on the developer's `@handle` hub page.
- **FR-R06 (Subscription Logistics & Dedicated Crew)**: System MUST support a **Subscription Transport & Courier Model** for operational workforce. Clients MAY subscribe to dedicated personal drivers, couriers, or language-matched escorts (e.g. Russian, English, Turkish, Georgian) for a fixed monthly subscription quota ($N$ hours or $M$ trips). Usage exceeding the quota MUST be automatically billed at the worker's set overage rate via `undrepay` / Paddle. Workers MUST be able to cap max active subscribers (default: 5 per driver) to prevent scheduling collisions.

---

## 3. Key Entities

- **DeveloperProfile**: `@handle`, GitHub ID, verified skills array, mode A/B ratings, completed bounties count.
- **DedicatedCrewSubscription**: `id`, `client_user_id`, `crew_provider_id`, `service_type` (`driver` | `courier` | `escort`), `monthly_quota_units`, `used_units`, `overage_rate`, `languages[]`, `status` (`active` | `paused` | `canceled`).
- **BountyTask**: `id`, GitHub issue URL, `amount`, `currency` (USDT/TON), `status` (`open` | `in_review` | `paid` | `disputed`), `escrow_intent_id` (`upi_...`).

---

*End of Specification.*
