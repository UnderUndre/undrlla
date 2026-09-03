# Tasks: 003-housing-donations

## Tasks

1. [BE] H-T1 — Housing extension schema & ServiceArea collection
   - Owner: `backend-specialist`
   - Est: 3d
   - Deliverables: `ServiceArea`, `HousingProgram`, `HousingApplication`, `HousingEscrow`, `HousingPoolAllocation`, `HousingPool` collection definitions with FLS & RLS policies per `data-model.md`.

2. [BE] H-T2 — Intake country gating, amount policy & KYC/AML transition guards
   - Owner: `backend-specialist`
   - Est: 3d
   - Depends on: H-T1
   - Deliverables: Intake validation hook rejecting disabled countries (`country_not_enabled`); `requested_amount` self-serve auto-approve vs over-cap `pending` (default cap 500000 USD) and allocation block until amount approved; admin amount-approve/reject; duplicate application detector; applicant `kyc_status` and destination `destination_kyc_status` transition guards.

2b. [BE] H-T2b — HousingPoolSubscription + 5% fee + ledger + queue model B
   - Owner: `backend-specialist`
   - Est: 3d
   - Depends on: H-T1
   - Deliverables: `HousingPoolSubscription` CRUD + **Paddle** (pre-LLC) / Stripe (post-LLC) / crypto recurring; FR-024 fee split on donation Orders; `HousingPoolLedgerEntry`; queue_entered_at / active|paused|left (7d grace, 90d leave); chargeback reverse path FR-023; applicant queue position read API. Fiat SoT = undrlla (not Medusa); see ECOSYSTEM D7 / Q10.

3. [BE] H-T3 — HousingEscrow destination snapshot & HousingPool credit hook
   - Owner: `backend-specialist`
   - Est: 3d
   - Depends on: H-T1
   - Deliverables: Immutable destination account reference snapshot at escrow creation, EUR/USD/USDT hard-currency held amount, Directus event hook crediting `HousingPool` on completed donation `Order` items.

4. [FE] H-T4 — Storefront & Telegram Mini App housing flows
   - Owner: `frontend-specialist`
   - Est: 3d
   - Depends on: H-T1, H-T2
   - Deliverables: Applicant apply/status/document-upload UI and concierge service booking integration in `undrllanding` storefront and TG Mini App.

5. [SEC] H-T5 — PII audit log redaction & contract e2e tests
   - Owner: `security-auditor`
   - Est: 3d
   - Depends on: H-T1, H-T2, H-T3
   - Deliverables: PII redaction middleware for `NotificationDelivery.last_error` and audit logs, contract tests for `/housing/*` endpoints, and e2e audit chain verification tests.

## Total estimate: 15 dev-days
