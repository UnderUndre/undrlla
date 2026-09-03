# Implementation Plan: 002-provisioning-portal

**Branch**: `spec/002-provisioning-portal` | **Spec**: `spec.md`  
**Updated**: 2026-07-30 — Medusa pivot hygiene (Directus shop provision narrative removed)

## Summary

Deliver an **operator-first** provisioning portal that emits a validated **`ProvisioningManifest` v2 (Medusa)** for `undevops` to one-click deploy **client shops**: Medusa template-shop + Postgres + Redis + optional `undrllanding` storefront + Traefik/TLS.

**Not in scope:** provisioning Directus as the client commerce runtime (legacy). Flagship undrlla/Directus (IdP, housing, polity) is a separate stack.

**SoT contracts:**

| Artifact | Path |
|----------|------|
| JSON Schema | `contracts/medusa-provisioning-manifest.schema.json` |
| Example | `contracts/medusa-manifest.example.json` |
| undevops ingest | `undevops/specs/003-undrlla-one-click-deploy` |
| Template app | `undreseller/specs/001-template-shop` |

## Technical Context

- **Portal host**: undrlla flagship API (Directus extension and/or slim REST) — **orchestration UI only**
- **Runtime provisioned**: **Medusa 2.0** (`undreseller` template-shop image), not Directus shop
- **Dependencies**: `undevops` HMAC ingest + secret store; domain lock; Paddle/Medusa env as **secret refs** only
- **MVP mode**: operator-only (customer self-service later)
- **Testing**: schema validation, domain lock, HMAC fixture, contract test against sample Medusa manifest, retry/DLQ

## Milestones

1. **Medusa Manifest Schema & Data Model** (2d): Persist `ProvisioningManifest` (`manifest_version: 2.0-medusa`), `ManifestAuditLog`, `DomainReservationLock`; validate against `medusa-provisioning-manifest.schema.json` (includes `list_on_hub`, `share_catalog_to_hub`, `domain`, `initial_admin_email`, `addons`, secret refs).
2. **Secret refs & domain lock** (2d): Pre-ingestion check against `undevops` secret store; atomic domain reservation; reject raw secrets.
3. **Operator review & RBAC** (2d): `requires_review` workflow; `provisioning:request` / `provisioning:approve`; audit comments.
4. **Ingest client + retry/DLQ** (2d): POST signed manifest to undevops; poll/webhook status; exponential backoff; DLQ + alert.
5. **Contract tests & operator runbook** (2d): OpenAPI + fixture from `medusa-manifest.example.json`; runbook for Gate 0 (DNS, secrets, Medusa health).

**Total Estimate**: 10 dev-days

## Out of scope (this plan)

- Day-2 lifecycle (upgrade/suspend/destroy) — undevops future feature  
- Directus multi-tenant client shop snapshot provision — **deprecated**  
- Building the Medusa app itself — `undreseller`  
- Building undevops job code — `undevops/003`