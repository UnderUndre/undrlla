# Phase 0 Research: 003-housing-donations (undrlla-flagship, Tier-B)

**Spec**: `spec.md` | **Data Model**: `data-model.md` | **Date**: 2026-07-21

Record of the key architectural decisions taken for `003`, with rationale and rejected alternatives. These mirror the trade-offs captured in the spec Clarifications and Edge Cases.

## D1 — Housing as first-class collections, not a `need_category` in MAQ

**Decision**: Model housing as first-class collections (`HousingProgram`, `HousingApplication`, `HousingEscrow`, `HousingPoolAllocation`, `HousingPool`) rather than as one `need_category='housing'` entry inside the deferred $300k Mutual Aid Queue.

**Rationale**: Property purchase has lifecycle stages, KYC/AML gates, destination snapshotting, FX-aware escrow holds, and concierge-service linkage that a generic queue entry cannot express cleanly. Forcing housing into a queue row would either bloat the queue schema with nullable housing-only fields or push domain logic into unstructured JSON. First-class collections keep the audit chain explicit and let `003` ship before any MAQ spec exists.

**Alternatives rejected**:
- *`need_category='housing'` in MAQ*: requires the MAQ spec to exist first; couples two flagship modules; forces housing-specific fields onto every queue row.
- *Generic "AidApplication" collection with a discriminator*: speculative generalization before a second aid category is actually needed. Revisit if/when MAQ lands.

**Consequence / open point**: D4 below — what happens to `HousingPool` when a future MAQ spec is authored.

## D2 — Universal `ServiceArea` collection, not per-feature `enabled_countries`

**Decision**: One `ServiceArea` collection (country, optional region, `feature_scope`, `enabled`, notes) serving as the single admin-panel source of truth for "where is feature X available". Reused by `housing_purchase` and `concierge_service` in `003`, and reserved for `delivery`, `taxi`, `warehouse`, `barter`, `marketplace_directory` in future Tier-B specs.

**Rationale**:
- One admin panel for all geo-scoped features (no per-feature country editors drifting out of sync).
- Per-tenant scoping possible via nullable `tenant_id` (operator-global for `housing_purchase`, vendor-scoped for `concierge_service`).
- Extension-friendly: a future Tier-B feature adds a new `feature_scope` enum value, not a new collection.

**Alternatives rejected**:
- *Per-program `HousingProgram.enabled_countries` array*: minimally simpler for housing alone, but duplicates the country list across features and risks TR enabled for housing but disabled for concierge by accident. Also weak for region-level scoping.
- *Per-vendor `Service.geos`*: pushes scoping into vendor hands; no central operator control. Acceptable as a complement, not a replacement.

**Consequence**: `ServiceArea` is the first `003` collection a reviewer should look at — its design sets the pattern for every future Tier-B geo-scoped feature.

## D3 — Founder as single-vendor concierge in MVP; data model vendor-parameterized

**Decision**: In MVP, the founder is the sole approved concierge provider (`provider_id` = founder under the founder `tenant_id`). The four concierge offerings are ordinary `Service` rows. The schema is vendor-parameterized — adding a second approved provider post-MVP is a data change, not a schema change.

**Rationale**:
- Concierge quality is the differentiator; multi-vendor marketplace dynamics (ratings, arbitration, vetting) are not the problem to solve in MVP.
- Reusing Tier-A `Service`/`Booking`/`PaymentAttempt` means **zero new collections** for concierge — the entire `001` booking/notifications/refunds machinery applies for free.
- A single-vendor MVP also makes operational risk concrete (founder availability) rather than hiding it behind multi-vendor abstraction.

**Alternatives rejected**:
- *Multi-vendor concierge marketplace from day one*: forces vetting, ratings, arbitration, and dispute UI into MVP — well beyond the donation-housing value proposition.
- *Dedicated `ConciergeService` collection*: unnecessary duplication of `Service`; would split booking/notifications/refund logic between two near-identical code paths. Strongly rejected.

**Consequence**: Single-vendor outage risk is real in MVP. Mitigations: the standard `Booking` cancel/reschedule flow (FR-019 of `001`) applies, and post-MVP multi-vendor is a data change. Post-MVP providers include housing/CBI workforce roles (lawyers, realtors, translators, escorts) per `001` FR-039 and `003` FR-019 — still ordinary `Service` rows, price-first discovery.

## D4 — `HousingPool` separate from MAQ (open dependency, deferred decision)

**Decision**: `003` introduces `HousingPool` as a **housing-only** donation pool, independent of the $300k Mutual Aid Queue described (and deferred) in `001` FR-009..FR-011. When a future MAQ spec (`004+`) is authored, the MAQ owner chooses one of:

(a) **Merge** — `HousingPool` becomes a specialization of the general MAQ pool, tagged `need_category='housing'`, with the housing-specific escrow chain preserved.
(b) **Stay separate** — `HousingPool` remains a dedicated pool with its own audit/escrow chain; MAQ is for non-housing aid only.

Either resolution is acceptable. **The choice is explicitly deferred to the MAQ spec author.**

**Rationale**: `003` must be implementable today. Waiting for an MAQ spec that does not exist would block the housing feature indefinitely. Introducing a housing-only pool now is reversible (merge later) or stable (stay separate); either path is cheaper than blocking.

**Alternatives rejected**:
- *Block `003` until MAQ spec lands*: violates the SpecKit principle that each feature ships independently when it can.
- *Define MAQ in `003`*: scope creep; MAQ has its own complexity (queue scoring, activity accelerators, Pressure Points / Flow) that has nothing to do with housing.

**Open question for the MAQ spec author**: does merging require migrating existing `HousingPool` balances and `HousingPoolAllocation` history into the MAQ ledger, or is the MAQ ledger append-only and `HousingPool` stays as-is going forward? Flag this in the MAQ spec's own Clarifications when authored.

## D5 — Three funding models; Model A only in MVP

**Decision**: Recognize three funding models for housing (A: pool → escrow; B: personal donor balance; C: direct payment + commission). **Only Model A is in scope for `003`.** Models B and C are described in the spec Executive Context and User Stories 3 and 4 for traceability but MUST NOT be implemented (spec FR-009 / FR-010 SCOPE-Deferred block, SC-010 / SC-011 deferred).

**Rationale**:
- Model A is the simplest model that delivers the core value (verified housing reaches a real person via escrow, no raw cash to recipient) and aligns directly with the `001` FR-011 escrow-payout principle.
- Model B requires a `DonorLedger` / `DonorBalance` model, spend authorization, contribution-source AML checks, and anti-abuse rules — a separate sub-system.
- Model C competes with existing real-estate-escrow SaaS and is not the differentiating value of a donation platform.

**Alternatives rejected**:
- *Model B in MVP*: scope creep; personal-balance ledger is its own design problem.
- *Model C in MVP*: commercial ambiguity; not the platform's core identity.

## D6 — Platform default USD 500k; self-serve ≤ cap; over-cap amount approval

**Decision**: Flagship product is a **single package** — housing + citizenship-by-investment (`program_type=housing_cbi`). Passport is not a separate donation goal. Platform default `target_amount` / `self_serve_max_amount` = **USD 500,000**. Applicants may choose any positive `requested_amount` ≤ the self-serve cap without amount-admin approval; amounts **above** the cap require admin `amount_approval_status=approved` before pool allocation. Country CBI legal minima (e.g. Türkiye real-estate route ~USD 400k) are stored as optional `advisory_cbi_min` (warn-only in MVP, not a hard block).

**Rationale**:
- One number is easier to market and implement than multi-tier residency vs CBI defaults.
- USD 500k ≈ common CBI property floors (e.g. TR 400k) + fees/buffer; lower self-serve amounts fill faster (user preference).
- Unbounded self-serve high targets risk pool drain, abuse optics, and AML review load — over-cap needs a human gate.
- Separating amount-approval from KYC verify keeps "how much" and "who is this person" as independent controls.

**Alternatives rejected**:
- *Hard-code per-country legal minima as required floors*: country rules change; some applicants want housing without CBI; MVP would couple product to legal research that goes stale.
- *Always require admin approval for any amount*: kills the "fill faster under 500k" product goal.
- *Separate citizenship vs passport campaigns*: confuses donors; passport is an outcome of citizenship, not a second product.

**Consequence**: Allocation/escrow use `HousingApplication.requested_amount`. Post-`003`: rental PM and smart-home stay deferred (FR-018).

## D6b — Housing country ⟂ papers track country (2026-07-27)

**Decision**: An application MAY target **housing purchase** in country **X** and a **residency / CBI / naturalization track** in country **Y**. The system MUST NOT force `housing_country == papers_track_country`. Donation pool + escrow fund **housing** only; papers track is **advisory metadata + optional concierge/checklist**, never a second donation goal or a passport guarantee.

**Fields (see data-model)**:
- `HousingApplication.housing_country` — gated by `ServiceArea` `feature_scope='housing_purchase'` (same hard gate as legacy `country`).
- `HousingApplication.papers_track_country` — nullable ISO / M2O `ServiceArea` with `feature_scope='papers_track'`; may differ from housing.
- `papers_track_type`: `none` | `cbi_re` | `residency` | `naturalization` | `other`.
- Optional `papers_same_as_housing` UX default `true` (copies housing → papers on create; user can decouple).

**Rationale** (CONSTITUTION §VII.2; dialog 2026-07-27):
- Real users buy RE in one jurisdiction and pursue papers elsewhere (or housing-only).
- Coupling X=Y in schema encodes a false legal product and blocks legitimate plans.
- Hard-gating papers country would block housing when research backlog has papers track `enabled=false`.

**Alternatives rejected**:
- *Force same country*: simpler UI; lies about law and user intent.
- *Two donation goals (housing vs passport)*: already rejected in D6 / clarifications.
- *Hard-block papers_track when ServiceArea disabled*: too aggressive; use **warn-only** for papers; **hard-block only housing_purchase**.

**Consequence**: Legacy field `HousingApplication.country` / `HousingProgram.country` = **housing** country (alias documented). Intake FR-002 remains housing-only. Copy: no “guaranteed citizenship”. MVP live housing: TR+GE; papers_track rows may exist as research (`enabled` free for ops).

## D6c — Housing Geo Destinations & Real Estate Selection Engineering (2026-08-16)

**Decision**: While legacy MVP seed configuration included `TR` (Turkey) and `GE` (Georgia) in `ServiceArea`, the canonical recommended Tier-1 expansion destinations for donation-funded housing & papers tracks are:
1. **Cyprus (`CY`)**: €300,000 Fast-Track Permanent Residency (Cat 6.2) + Non-Dom 0% dividend/interest tax system + Common Law.
2. **United Arab Emirates (`AE`)**: AED 2,000,000 Golden Visa (10-year residency) + 0% personal tax + USD currency peg (3.6725 AED). (Operational directive: strict Tier-1 developer filtering for Tasreef drainage compliance post-2024 flood).
3. **Panama (`PA`)**: $300,000 Qualified Investor Visa (Decree 193) + USD economy + 0% territorial tax on foreign income + below карибских ураганов belt.
4. **European Continental Destinations (`GR`, `MT`, `IT`, `ME`)**:
   - **Greece (`GR`)**: €250,000 Commercial-to-Residential Conversion Golden Visa (Law 5100/2024; 5-year renewable visa, 0 days residence req, 50% tax relief Art. 5C; Airbnb prohibited).
   - **Malta (`MT`)**: €375,000 Permanent Residency (MPRP) + Non-Dom remittance-basis tax (0% offshore tax) + English law.
   - **Italy (`IT`)**: Residenza Elettiva via RE purchase + €200k/yr global Flat Tax (Art. 24-bis) or 7% Southern Italy Flat Tax (Art. 24-ter).
   - **Montenegro (`ME`)**: Any residential property (from €60k) = annual renewable Boravak (EUR currency, 9-15% tax, top EU accession candidate 2028-2030).

**Primary Residence (Owner-Occupant) US & Canada Expansion Targets**:
1. **Canada — Alberta (`CA-AB`, Calgary/Edmonton)**: 🏆 Top #1 Canada destination. 0% PST, 0% Land Transfer Tax, healthy P/E (3.5x-4.5x). *(Warning: Foreign Buyer Ban active until Jan 1, 2027; Work Permit holders require ≥183 days. Avoid Ontario `CA-ON` 25% NRST + double LTT and BC `CA-BC` 20% Foreign Buyer Tax).*
2. **USA — North Carolina (`US-NC`, Research Triangle)**: 🏆 Top East Coast US destination. Flat ~4.5% income tax, low ~0.8% property tax, strong tech hub.
3. **USA — Washington State (`US-WA`, Bellevue/Redmond/Seattle Eastside)**: 0% State Income Tax, ~0.95% property tax, Big Tech capital accumulation.
4. **USA — Nevada (`US-NV`, Reno/Summerlin) & Tennessee (`US-TN`, Nashville)**: 0% State Income Tax, low ~0.55%-0.65% property tax.

**Advisory Warning Gates & Blacklists**:
- **Turkey (`TR`)**: Flagged high risk due to Marmara fault seismic infrastructure collapse risk in Istanbul, SPK appraisal gap ($480k-$520k cash for $400k paper value), Akkuyu NPP proximity / lack of central gas (Doğalgaz) / closed districts (Kapalı Mahalleler) in Mersin.
- **Georgia (`GE`)**: `GE` is reserved primarily for 1% Small Business tax / OpCo structure (NACE 62.01). Property purchase in Batumi is flagged for high humidity/mold risk and illiquid secondary resale.
- **USA — Texas (`US-TX`) & Florida (`US-FL`)**: Flagged high risk. Texas has a brutal 1.8%–2.5%/yr property tax trap on market value ($16k–$20k/yr on $800k house); Florida has insurance crisis ($5k–$10k/yr) and SB 264 law restrictions for non-permanent residents within 10 miles of military/infra.
- **Spain (`ES`) & Portugal (`PT`)**: Hard-blocked / blacklisted (Spain Golden Visa abolished in 2025 + okupas law risk; Portugal RE Golden Visa abolished 2023 under Mais Habitação).
- **Germany (`DE`), France (`FR`), Netherlands (`NL`)**: Hard-blocked for holding (Schlüsselgewalt § 8 AO tax trap + EPBD energy renovation mandates).

## D7 — Shared pot + queue; 5% donation fee; earmark later (2026-07-22)

**Decision**: Primary story is housing+CBI for recipients, but **donors fund a shared `HousingPool`** (subscription-first, optional one-off). **Queue/admin selects** the next verified applicant. **No earmarked** peer donations in MVP. **Platform donation fee = 5%** (default) of gross donation; remainder credits pool; payment-rail fees pass-through by default. Earmarked gifts to a named person/company are a later feature.

**Rationale**:
- Shared pot is simpler ops/AML story than thousands of personal CBI crowdfunds.
- Subscription matches Mutual Aid monthly habit and predictable pool inflow.
- 5% is donor-tolerable and funds compliance/ops without looking extractive on charity-adjacent flows (contrast logistics 12%).
- Earmark later avoids progress-bar fraud and “why didn’t my person get funded” support load in MVP.

**Alternatives rejected**:
- *Personal crowdfund thermometers in MVP*: marketing-friendly but queue fairness and chargeback hell.
- *0% platform fee*: unsustainable for KYC/escrow ops.
- *12% on donations*: too aggressive for pot/subscription optics.

**Consequence**: Public UX is “subscribe to the housing+passport pot”, not “fund Vasya’s passport”. Applicant-facing UX still shows their `requested_amount` and queue status.

## D8 — Model B queue: active sub keeps seat (2026-07-22)

**Decision**:
- Sub: **≥ USD 1 / month**, custom amount; one-offs feed pot only.
- Queue **model B**: seat is **`active` only with live subscription**. FIFO `queue_entered_at = max(first_contribution, verified_at, subscription_activated_at)`.
- Grace: **7 days** charge retry → `paused` (keep key) → **90 days** → `left` (rejoin = new key). Resub < 90d → same key.
- Shared pot; no earmark; no neediness score; donations non-refundable (self-serve); chargebacks → unallocated balance (FR-023).
- Entities: `HousingPoolSubscription`, `HousingPoolLedgerEntry` (2026-07-23 review fix).

**Rationale**: Pot must grow toward ~$500k packages; free-rider “$1 once, wait forever” starves the kettle and clogs the line. $1/mo is a participation signal, not a means-test. Pause/90d avoids punishing a bad card for one cycle while still clearing zombies.

**Rejected**: Model A (one-time keeps seat forever) — simpler UX, worse economics and fairness under load.

**“Neediness”**: not a system field; only `verified`.

## Operational inputs and external dependencies

- **`001` machinery reuse**: `Service`, `Booking`, `PaymentAttempt`, `Order`, `OrderSplit`, `NotificationDelivery`, the payment-provider abstraction, and the escrow principle of FR-011 are all consumed unchanged. `003` adds collections; it does not modify `001`.
- **Directus extension packaging**: all `003` collections ship in one isolated extension package registered via the FR-016 hook point. A clean Tier-A instance boots without it (spec FR-014, SC-008).
- **KYC/AML provider (out of scope, legal dependency)**: real-property transactions exceed typical AML thresholds in both TR and GE. KYC of the applicant, source-of-funds checks on large donor contributions, and verification of the notary / escrow agent identity are operational and legal prerequisites. `003` requires only that the `kyc_status` / `destination_kyc_status` flags exist and be required before the corresponding transitions (spec FR-012). Integration with a specific provider (e.g. Onfido, Sumsub, Persona) is **out of scope** for `003` and deferred to a dedicated KYC integration spec or to operational runbook configuration.

## Operational risks flagged

- **Exchange-rate volatility on long escrow holds (Turkish lira)**: a `HousingEscrow` may sit in `held` status for weeks while a property signing is scheduled. TRY is historically volatile. `HousingEscrow.settlement_rate_snapshot` records the rate at creation for transparency, but `003` MVP does **not** implement automatic re-pegging or FX-hedge logic. Long holds should be operationally minimized; flag this in the operational runbook (tasks phase).
- **Single-vendor concierge outage**: see D3. Mitigated by standard `Booking` cancel/reschedule; multi-vendor is post-MVP.
- **Country disabled mid-application**: see spec Edge Cases. Application parks at current status with `country_disabled` flag.
- **Notary / escrow agent rejects signing**: `HousingEscrow` transitions to `failed_release`; funds return to `HousingPool` by default (configurable policy). Application reverts to `approved` for re-scheduling.
- **Duplicate / fraudulent applications**: detected by `(applicant_user_id, target_property_ref)` index and surfaced to admin before verification.

## Open questions (non-blocking, for future specs or operational runbook)

1. **MAQ merge** — does `HousingPool` merge into the future MAQ, or stay separate? (D4; deferred to MAQ spec author.)
2. **Partial pool allocations** — when `HousingPool.balance` is insufficient for a full `target_amount`, should `003` support partial allocations and progress-based release? (Spec FR-006 currently says MVP is full-amount only; revisit if real operations demand partial.)
3. **KYC/AML provider integration** — which provider, and is it shared with the `001` vendor-onboarding KYC (Stripe Connect Express) or separate? (Out of scope for `003`; legal-dependency.)
4. **Multi-vendor concierge / housing-pro onboarding flow** — vetting, ratings, arbitration UI for lawyers, realtors, translators, and additional concierge providers. (Post-MVP; `003` FR-019.)
5. **Region-level scoping** — does `ServiceArea.region` need a structured taxonomy (ISO 3166-2) or is freeform acceptable for MVP? (Currently freeform; revisit if concierge area matching needs precision.)
6. **Hard-enforce `advisory_cbi_min`** — should MVP later refuse CBI-labeled campaigns below the country legal minimum, or stay warn-only forever? (Currently warn-only.) — apply advisory against **`papers_track_country`** (or housing when papers_same_as_housing), not a second fund goal.
7. **Platform commission on donations** — rate(s) not fixed in `003`; revisit when Model C / fee productization is specified.
8. **Post-`003` rental PM + smart home** — tenant rent in-app, owner split, optional smart locks (explicitly deferred; FR-018).
9. **Papers-track catalog depth** — how rich is `papers_track` ServiceArea metadata (min investment, residency days, dual-citizenship allowed)? Start with notes JSON; structured program catalog later.
