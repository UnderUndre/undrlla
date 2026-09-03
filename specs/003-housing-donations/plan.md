# Implementation Plan: 003-housing-donations

**Branch**: `specs/003-housing-donations` | **Spec**: `spec.md` (same folder)

## Summary

Deliver Tier-B flagship donation-funded housing purchase (Model A) and country-scoped concierge services. Flagship package is **housing + CBI** with default target **USD 500k**; applicant `requested_amount` ≤ self-serve cap needs no amount approval; over-cap requires admin amount approval. Rental PM and smart-home are post-`003`.

## Technical Context

- **Platform**: Directus 12.x extension package (`packages/extensions/housing-donations/`)
- **Dependencies**: `001-init-from-directus` (reuses `Service`, `Booking`, `PaymentAttempt`, `Order`, `OrderSplit`)
- **Testing**: End-to-end audit chain tests, destination snapshot immutability tests, country gating tests, KYC/AML transition guards.

## Milestones

1. **ServiceArea & Housing Core Schema** (3d): Implement `ServiceArea`, `HousingProgram` (incl. `target_amount`/`self_serve_max_amount`/`advisory_cbi_min`), `HousingApplication` (incl. `requested_amount`/`amount_approval_status`), `HousingEscrow`, `HousingPoolAllocation`, `HousingPool` collections.
2. **Intake, amount policy & KYC/AML Guard Extension** (3d): Build country gating hook, self-serve amount auto-approve / over-cap pending gate, duplicate application detector, and applicant/destination KYC verification status transitions.
3. **Escrow Destination Snapshot & Event Hook** (3d): Implement destination snapshot freezing, `HousingPool` event listener for donation orders, and hard-currency escrow release.
4. **Concierge Geo-Link & UI Client Delta** (3d): Link concierge `Service` to `ServiceArea`, extend Next.js storefront & TG Mini App contracts.
5. **PII Sanitization, Contract Tests & CI** (3d): Implement audit log PII redaction filters, contract tests, and e2e test suite.

**Total Estimate**: 15 dev-days
