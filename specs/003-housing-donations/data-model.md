# Data Model: 003-housing-donations (undrlla-flagship, Tier-B)

**Spec**: `spec.md` (same folder) | **Builds-on**: `001-init-from-directus/data-model.md` | **Directus**: 12.x (v12.1.1)

This is the Tier-B flagship data model that populates the FR-016 extension-point seam shipped by `001-init-from-directus`. Collections map 1:1 to Directus collections; relations use Directus M2O/O2M/M2M fields. Access is enforced by Directus **policy/permission row filters** and **field permissions** (FLS) inherited from the `001` model — never by application code alone.

All collections in this file ship inside **one isolated Directus extension package** registered only by the flagship instance. A clean Tier-A instance contains none of them by default (spec FR-014, mirrors `001` FR-016 User Story 1 Acceptance Scenario 2).

## Tenancy & Policy Model (inherits `001`)

- **Instance isolation**: unchanged from `001` D1 — each client marketplace is its own Directus instance/container; the flagship Undrlla.com is simply Instance #1 with the `003` extension registered.
- **Within-instance tenancy**: `tenant_id` continues to identify a sub-organization/storefront. All `003` collections are `tenant_id`-scoped for vendor-style isolation, with two explicit exceptions:
  - `ServiceArea` is **instance-global** (`tenant_id` nullable) when a feature is operator-scoped (e.g. `feature_scope='housing_purchase'` is a platform-wide decision, not per-vendor). It carries `tenant_id` only when a feature is tenant-scoped (e.g. a specific vendor's `concierge_service` area).
  - `HousingApplication.applicant_user_id` is **customer-scoped**: an applicant reads their own applications regardless of which `tenant_id` the program lives under (analogous to the customer-scoped cross-tenant read model in `001` FR-031).
- **FLS**: sensitive columns (`HousingApplication.proof_urls`, `HousingApplication.target_property_ref`, `HousingEscrow.destination_account_ref`, applicant identity document references) are stripped for Customer/Vendor policies per `001` FR-036, extended by `003` FR-011. Admins read full records for support/audit.
- **Indexes**: `tenant_id`, `country`, `feature_scope`, `enabled`, `applicant_user_id`, `program_id`, `status`, `source_pool_id`.

## Sensitive data handling (spec FR-011, extends `001` FR-036)

- Property addresses, applicant identity documents, notary / escrow-agent identities, and destination bank-account / wallet references are sensitive operational PII.
- API input uses structured fields (`AddressInput` from `001`, plus `DestinationAccountRef` discriminated union: `{type: 'iban', iban, bic}` | `{type: 'crypto_wallet', chain, address}` | `{type: 'vendor_ref', vendor_id}`) rather than untyped blobs. Validation happens at the `/housing/*` and `/admin/housing/*` boundaries before data reaches Directus collections.
- Field permissions mask or omit PII by role:
  - **Admin**: full read for support/audit.
  - **Applicant**: reads only their own application's address/proof/destination; never sees another applicant's data.
  - **Vendor / concierge provider**: reads only the PII strictly required to deliver a booked service (e.g. the escort-to-signing vendor sees the notary meeting address for bookings they confirmed; never sees the full bank-account destination or another applicant's identity docs).
- Audit logs and `NotificationDelivery.last_error` MUST redact raw document numbers, full IBANs, full wallet addresses, and full addresses — use stable record IDs and short suffixes for support correlation. Mirrors `001` FR-036 redaction rules.
- Retention is per instance: application proof / destination / address data is retained only for fulfilment, refund, tax, and statutory periods, then deleted or anonymized by scheduled maintenance.

## Roles → filter summary (extends `001`)

| Role | ServiceArea | HousingProgram | HousingApplication | HousingEscrow | HousingPool |
| ---- | ----------- | --------------- | ------------------ | ------------- | ----------- |
| Admin | all | all | all | all | all |
| Founder-vendor (MVP) | own `tenant_id` (concierge areas only) | own `tenant_id` | own `tenant_id` (for their programs) | own `tenant_id` | own `tenant_id` |
| Customer / Applicant | read enabled-only (public) | read active-only (public) | own (`applicant_user_id`) | own (via application) | none |
| Other vendor | own `tenant_id` (concierge areas only) | none | none | none | none |

## Collections

### ServiceArea  *(instance-global or tenant-scoped — spec FR-001)*

`id`, `country` (ISO 3166-1 alpha-2, indexed), `region` (nullable), `feature_scope` enum (`housing_purchase` | `papers_track` | `concierge_service` | reserved `delivery` | `taxi` | `warehouse` | `barter` | `marketplace_directory`), `enabled` boolean (indexed), `tenant_id` (M2O Tenant, nullable — nullable for operator-scoped features like `housing_purchase`), `notes`, `metadata` (nullable JSON — e.g. advisory min investment for a papers track), `updated_at`, `updated_by`. M2O → Tenant (nullable). This is the single admin-panel source of truth for "where is feature X available".

- **`housing_purchase`**: hard gate for property funded by the pool (MVP: TR+GE `enabled=true`).
- **`papers_track`**: orientation / checklist / concierge-for-docs availability for residency, CBI, naturalization, etc. **Warn-only** if disabled — does **not** block housing intake (research.md D6b).

### HousingProgram  *(spec FR-003)*

`id`, `tenant_id` (M2O Tenant), `country` (M2O ServiceArea with `feature_scope='housing_purchase'` — **housing** country for the program template; alias: `housing_country`), `default_papers_track_country` (nullable M2O ServiceArea with `feature_scope='papers_track'` — template default only; applicants MAY override to a different country), `default_papers_track_type` enum/string (`none` | `cbi_re` | `residency` | `naturalization` | `other`; default `cbi_re` for `program_type=housing_cbi`), `program_type` enum/string (flagship default type: `housing_cbi` — single **package story** "housing + path-to-papers orientation", not separate citizenship/passport **donation goals**; other types e.g. `housing_need`, `residency_only` allowed), `target_amount` (decimal; platform default **500000** for `housing_cbi`), `currency` (default `USD`), `self_serve_max_amount` (decimal; default **500000** — applicants may set application `requested_amount` ≤ this without amount-admin approval), `advisory_cbi_min` (nullable decimal — advisory legal minimum for the **default papers** track when applicable — warn-only; does not hard-block funding), `eligibility_rules` (structured JSON: min/max purchase price, applicant residency, income rules, etc.), `payout_destination_type` enum (`notary` | `escrow_agent` | `verified_vendor`), `status` (`draft` | `active` | `paused` | `closed`). O2M → HousingApplication. A `HousingProgram` MUST NOT be `active` while its **housing** `ServiceArea.enabled=false` (enforced by extension validation). Papers-track ServiceArea may be disabled without forcing program `paused`.

### HousingApplication  *(spec FR-004, FR-017, FR-022, FR-025 housing⟂papers)*

`id`, `program_id` (M2O HousingProgram), `applicant_user_id` (M2O Directus user — customer-scoped), `country` (**housing** country; denormalized from program / `housing_country` for intake filtering — **hard-gated** by `housing_purchase` ServiceArea), `housing_country` (M2O ServiceArea `feature_scope='housing_purchase'`; MUST match `country` or replace it in implementations — same semantic; dual name for clarity in CONSTITUTION language), `papers_same_as_housing` (boolean, default `true`), `papers_track_country` (nullable M2O ServiceArea `feature_scope='papers_track'` **or** ISO alpha-2 string + optional area id — MAY differ from housing country), `papers_track_type` enum (`none` | `cbi_re` | `residency` | `naturalization` | `other`), `papers_track_warn` (nullable string/json — e.g. `papers_track_not_enabled`, `below_advisory_min`, `cross_border_complexity`), `status` enum (`draft` | `submitted` | `verified` | `approved` | `funded` | `escrow_released` | `completed` | `rejected`), `requested_amount` (positive decimal — need size when allocated; allocation and escrow use this value for **housing**; **not** an earmarked crowdfund goal; **not** a second passport fund), `amount_approval_status` enum (`auto_approved` | `pending` | `approved` | `rejected`), `amount_approved_at` (nullable), `amount_approved_by` (M2O admin user, nullable), `queue_entered_at` (nullable datetime — FIFO key per FR-022 model B), `queue_status` enum (`ineligible` | `active` | `paused` | `funded` | `left`), `queue_paused_at` (nullable), `first_pool_contribution_at` (nullable), `subscription_activated_at` (nullable — when housing-pool sub first became active for queue math), `proof_urls` (FLS — array of uploaded proof assets: identity, income, need documentation), `target_property_ref` (nullable, FLS — address or agency listing reference; property is in **housing** country), `kyc_status` enum (`pending` | `passed` | `failed`), `duplicate_flag` (nullable — surfaced to admin before verification), `submitted_at`, `verified_at`, `verified_by` (M2O admin user), `approved_at`, `funded_at`, `escrow_released_at`, `completed_at`, `rejected_at`, `rejection_reason`. O2M → HousingPoolAllocation. O2M → HousingEscrow (typically one). Indexed on `(applicant_user_id, target_property_ref)` for duplicate detection; indexed on `(queue_status, queue_entered_at)` for allocation; indexed on `papers_track_country` for ops reporting.

**Housing ⟂ papers rules:**
- On create: if `papers_same_as_housing=true`, set `papers_track_country` from housing country’s ISO (and default type from program) unless applicant overrides.
- Applicant MAY set `papers_same_as_housing=false` and choose another papers country/type.
- Intake **rejects** only when **housing** country has no enabled `housing_purchase` ServiceArea (`country_not_enabled`).
- If papers track is missing/disabled/below advisory min → set `papers_track_warn`; **do not** reject housing application solely for that reason.
- Escrow destination and property ref are always in the **housing** jurisdiction.

**Amount policy (enforced in extension validation):**
- On create/update of `requested_amount`: if `requested_amount` ≤ linked program `self_serve_max_amount` → `amount_approval_status = auto_approved`; if greater → `amount_approval_status = pending` (admin must set `approved` or `rejected`).
- Raising `requested_amount` across the self-serve cap resets status to `pending`.
- `HousingPoolAllocation` creation is forbidden unless `amount_approval_status` ∈ {`auto_approved`, `approved`} and application is `verified` (or later allowed states per FR-004).

### HousingEscrow  *(spec FR-005 — destination-snapshot pattern)*

`id`, `application_id` (M2O HousingApplication), `destination_type` enum (`notary` | `escrow_agent` | `verified_vendor`), `destination_account_ref` (FLS — snapshotted structured reference: IBAN+bank_name / crypto chain+address / vendor_id; FROZEN at creation), `destination_kyc_status` enum (`pending` | `passed` | `failed`), `amount_held`, `currency`, `settlement_rate_snapshot` (nullable — rate vs canonical currency at creation, for long-hold FX transparency), `status` enum (`held` | `released` | `failed_release`), `snapshot_at`, `released_at`, `released_by` (M2O admin user), `failure_reason`. A destination change after creation applies only to future escrows; existing `destination_account_ref` is immutable for the life of the escrow (mirrors `PaymentAttempt.payout_wallet_snapshot` from `001` FR-025). Funds move out only on a `provider_confirmed`-equivalent signal (notary / verified escrow agent / verified vendor confirms signing/handover).

### HousingPoolAllocation  *(spec FR-006)*

`id`, `application_id` (M2O HousingApplication), `source_pool_id` (M2O HousingPool), `amount`, `currency`, `allocated_at`, `allocated_by` (M2O admin user, nullable for auto-allocation). Creating a row MUST atomically debit `HousingPool.balance` (extension-enforced). MVP supports full-amount allocation only (one allocation per application covering the program's `target_amount`); partial allocations are deferred.

### HousingPool  *(spec FR-007 — housing-only in `003`)*

`id`, `tenant_id` (M2O Tenant), `balance`, `currency`, `status` enum (`active` | `paused` | `closed`), `created_at`, `updated_at`. The balance MUST be debited only through a `HousingPoolAllocation` and credited only through auditable incoming donation events (donation events themselves are represented as `PaymentAttempt` records in the `001` model with `Order`/`OrderSplit` linkage to a dedicated donation product, or — if the future MAQ spec introduces a dedicated donation ledger — migrated there; see spec Executive Context open dependency and `research.md` D4). Housing-only in `003`. Chargebacks: FR-023 (debit unallocated balance; pause pool if insufficient).

### HousingPoolSubscription  *(spec FR-021 — model B queue)*

`id`, `user_id` (M2O Directus user), `pool_id` (M2O HousingPool), `amount` (decimal ≥ 1.00), `currency` (default USD), `interval` enum (`month` default), `status` enum (`active` | `past_due` | `canceled` | `paused`), `provider` (`paddle` pre-LLC default | `stripe` post-LLC | `ton` | undrepay crypto ids | …), `provider_subscription_ref` (nullable), `activated_at`, `canceled_at`, `grace_until` (nullable — failed charge grace end), `created_at`, `updated_at`. One active sub per user per pool in MVP. Status transitions drive `HousingApplication.queue_status` (FR-022). Fiat SoT remains undrlla (not Medusa); see ECOSYSTEM D7/Q10.

### HousingPoolLedgerEntry  *(spec FR-024 / FR-023)*

`id`, `pool_id`, `user_id` nullable, `type` enum (`donation_credit` | `fee` | `allocation_debit` | `chargeback_reversal` | `adjustment`), `gross_amount`, `fee_amount`, `net_amount`, `currency`, `payment_attempt_id` nullable, `application_id` nullable, `created_at`. Append-only audit of pot movements.

### Concierge services  *(spec FR-008 — NO new collection)*

The four MVP concierge offerings (escort-to-signing, proxy-purchases via local bank card, rental-handover help, legal/banking setup) are ordinary `Service` rows from `001`:

- `provider_id` = founder's user id (MVP; any future approved vendor post-MVP).
- `tenant_id` = founder's tenant.
- `payment_policy` = `prepaid` or `pay_later_free` per offering.
- `availability` = slot rules (calendar of when the founder/vendor is available in-country).
- Optional geo-scoping: a `Service` MAY be linked to one or more `ServiceArea` rows with `feature_scope='concierge_service'` (M2M via a join collection or a typed tag; the extension enforces `service_not_available_in_area` on bookings outside the linked areas). A service with no `ServiceArea` link is bookable globally.

Concierge bookings flow through the standard `Booking` (FR-019/FR-030 of `001`), pre-paid ones create a standard `PaymentAttempt`, and refunds follow FR-020 of `001`. No concierge-specific collection, trigger, or hook ships in `003`.

## Key relations

- Tenant has many ServiceArea (where `tenant_id` is set), HousingProgram, HousingPool, and (via founder-as-vendor) concierge Services.
- HousingProgram has many HousingApplication records; HousingApplication has one HousingProgram.
- HousingApplication has one (MVP) HousingEscrow and one (MVP, full-amount) HousingPoolAllocation.
- HousingPool has many HousingPoolAllocation records; each HousingPoolAllocation links an application to its funding source pool.
- ServiceArea is referenced by HousingProgram.country / HousingApplication.housing_country (filter `feature_scope='housing_purchase'`), by HousingApplication.papers_track_country (filter `feature_scope='papers_track'`, soft), and optionally by concierge Services (filter `feature_scope='concierge_service'`).
- Customer/applicant reads span `tenant_id` (customer-scoped exception for `HousingApplication.applicant_user_id`), analogous to `001` FR-031.

## Tier-B deferred (NOT in `003`)

These are deliberately NOT modeled in `003`:

- `DonorLedger` / `DonorBalance` (Model B, personal donor balance — spec FR-009 deferred).
- Direct-payment housing `Order`-as-property-purchase with commission split (Model C — spec FR-010 deferred).
- The general `$300k` Mutual Aid Queue / `MutualAidQueue` collection (still owned by a future MAQ spec per `001` FR-009..FR-011; `003` ships only the housing-specific `HousingPool`).
- KYC/AML provider integration collections (spec FR-012 requires only the `kyc_status` / `destination_kyc_status` flags exist; provider integration is a legal-dependency logged in `research.md`).
- Ongoing rental property-management (leases, in-app rent payments, owner payout splits) and smart-home / keyless access (spec FR-018 — post-`003`).

## Migration / extension packaging

- All `003` collections ship in one isolated Directus extension package (e.g. `packages/extensions/housing-donations/`) with its own schema migration registered via the FR-016 hook point.
- Registering the extension is an explicit operator action; a clean Tier-A instance MUST boot and serve the `001` contract without this extension present (spec FR-014, SC-008).
- No `001` collection is altered by `003`. The `Service`, `Booking`, `PaymentAttempt`, `Order`, and `OrderSplit` collections from `001` are referenced read-only by concierge services and (optionally) by donation events that credit `HousingPool`.
