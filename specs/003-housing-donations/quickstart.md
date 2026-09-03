# Quickstart: 003-housing-donations (Tier-B) dev instance

**Spec**: `spec.md` | **Builds-on**: `001-init-from-directus/quickstart.md`

Run the donation-funded housing + concierge MVP locally on top of a `001` instance. This mirrors what registering the flagship extension does in production, minus ACME / domain automation.

This quickstart assumes you have already completed the `001` quickstart and have a running `undrlla-core` Directus instance at `http://localhost:8055` with Admin credentials in hand.

## Prerequisites

- A running `001` instance (see `001-init-from-directus/quickstart.md` steps 1–6)
- Admin credentials for that instance
- The founder's user id (the user who will be the sole MVP concierge `provider_id`) — note this id after step 4 below
- A canonical settlement currency chosen for the instance (e.g. `EUR`) — inherited from `001`
- Test payout-provider sandbox creds already configured for the `001` instance (Stripe test, TON testnet, SHKeeper test) — concierge pre-paid bookings reuse these unchanged

## 1. Build & register the `003` extension

The `003` collections ship as one isolated Directus extension package:

```bash
npm --prefix ./extensions/housing-donations ci
npm --prefix ./extensions/housing-donations run build
# register the extension in the instance's extension list
docker compose restart directus
```

After restart, Directus applies the `003` schema migration automatically. The new collections (`ServiceArea`, `HousingProgram`, `HousingApplication`, `HousingEscrow`, `HousingPoolAllocation`, `HousingPool`) appear in Directus Studio.

## 2. Enable TR and GE for housing purchase

In Directus Studio → `ServiceArea`, create two enabled records:

| country | region | feature_scope        | enabled | tenant_id |
| ------- | ------ | -------------------- | ------- | --------- |
| TR      | null   | `housing_purchase`   | true    | null      |
| GE      | null   | `housing_purchase`   | true    | null      |

Optionally add a third disabled record to verify intake rejection:

| country | region | feature_scope        | enabled | tenant_id |
| ------- | ------ | -------------------- | ------- | --------- |
| DE      | null   | `housing_purchase`   | false   | null      |

Smoke check (US1 AS1, US1 AS2): TR and GE are enabled; DE is disabled.

## 3. Seed the HousingPool and a HousingProgram

In Directus Studio:

- `HousingPool`: create one row under the founder's `tenant_id`, `currency=EUR`, `status=active`, `balance=0`. (Donate into it via a test `Order` in step 6, or set a test balance directly for the smoke test.)
- `HousingProgram`: create one row with `country` linked to the TR `ServiceArea`, `target_amount=50000`, `currency=EUR`, `payout_destination_type=notary`, `status=active`.

Smoke check (FR-003): the program cannot be `active` while its `ServiceArea.enabled=false` — toggle TR off and back on to verify the extension rejects the invalid state.

## 4. Seed the founder as concierge vendor + four concierge Services

Make the founder an approved vendor under the founder `tenant_id` (if not already from `001`). Note the founder's `user_id` — this is the concierge `provider_id` for MVP.

Create four `Service` rows (FR-008) under the founder tenant, each with `provider_id` = founder's user id:

| title                                | payment_policy     | ServiceArea link (optional)       |
| ------------------------------------ | ------------------ | --------------------------------- |
| Escort to property signing (TR)      | `prepaid`          | TR `concierge_service`            |
| Proxy purchases via local card (TR)  | `pay_later_free`   | TR `concierge_service`            |
| Rental-handover help (TR)            | `prepaid`          | TR `concierge_service`            |
| Legal / bank account setup (TR)      | `prepaid`          | TR `concierge_service`            |

Smoke check (US2 AS1, US2 AS2): all four appear in the catalog filtered to the founder tenant and are each independently bookable.

## 5. Submit a HousingApplication end-to-end (US1)

As the applicant (use a second test user, or impersonate):

1. **Apply** (US1 AS3): POST `/housing/applications` with `program_id`, proof URLs, `target_property_ref`. Verify status → `submitted`. Verify a DE application is rejected at intake with `country_not_enabled` (US1 AS2, SC-004).
2. **Verify** (US1 AS4): in Studio, an admin sets `kyc_status=passed` and transitions the application to `verified`. Verify it is now eligible for allocation.
3. **Approve + allocate** (US1 AS5): admin transitions to `approved`; a `HousingPoolAllocation` is created debiting the pool. Verify the pool balance dropped by `target_amount`.
4. **Escrow** (US1 AS6): admin creates a `HousingEscrow` with the notary's IBAN (test value). Verify the destination is snapshotted at creation, the amount is held, and the application transitions to `funded`.
5. **Release** (US1 AS7): simulate the notary confirming signing (admin-only hook in MVP). Verify funds move to the snapshotted destination (never to the applicant), the escrow → `released`, and the application → `escrow_released` → `completed`.

Smoke check (SC-002): the full audit chain `HousingPool → HousingPoolAllocation → HousingApplication → HousingEscrow → destination` is complete and contains no raw-cash transfer to the applicant.

## 6. Book a concierge service (US2)

As a customer:

1. Select the founder's "Escort to property signing (TR)" slot and request a booking (US2 AS3). Verify a `pending` `Booking` with a linked `PaymentAttempt` is created.
2. As the founder, confirm the booking. Verify payment is captured and the slot is atomically reserved.
3. (Negative test) Reject another overlapping booking. Verify the card authorization is voided or the crypto charge auto-refunded.
4. (Negative test) Try to book a TR-linked concierge service in a non-TR context. Verify refusal with `service_not_available_in_area` (US2 AS4, SC-006).

Smoke check (SC-005): all of the above uses only the `001` `Service`/`Booking`/`PaymentAttempt` machinery — no concierge-specific code path.

## 7. Verify (smoke checklist)

- [ ] `ServiceArea` admin toggle takes effect on intake immediately, no redeploy (SC-001).
- [ ] Full Model A audit chain complete, no raw-cash-to-applicant transfer (SC-002).
- [ ] `HousingEscrow` destination is snapshotted and immutable for the life of that escrow (SC-003).
- [ ] Disabled-country intake rejected; mid-application disable parks with `country_disabled` flag (SC-004).
- [ ] Four concierge services bookable via standard `Booking`, pre-paid captures on confirm (SC-005).
- [ ] Geo-scoped concierge booking refused outside its area (SC-006).
- [ ] Cross-applicant PII reads blocked; vendor sees only their own booked-service PII (SC-007).
- [ ] A clean Tier-A instance (extension unregistered) contains none of the `003` collections (SC-008) — verify by repeating the `001` quickstart without step 1 of this guide.

## 8. Cross-AI Review Gate reminder (Constitution Principle VI)

Before any implementation task begins (i.e. before writing `plan.md`, `tasks.md`, contracts, or code), the gate MUST be satisfied:

- `specs/003-housing-donations/reviews/analyze.md` exists with verdict `PASS` (or `OVERRIDDEN` with logged reason).
- At least two distinct external-AI reviewer files exist in `specs/003-housing-donations/reviews/<provider>.md` with verdict `PASS` (or `OVERRIDDEN` with logged reason). Same-provider twice counts as one.
- The spec author is NOT an external reviewer of this spec.

See `reviews/` for the analyze stub and the external-reviewer prompt template.

## Reset

```bash
docker compose down -v && docker compose up -d
# then re-run the 001 quickstart and step 1 of this guide
```
