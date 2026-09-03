# Feature Specification: Provisioning Portal (002)

**Feature Branch**: `spec/002-provisioning-portal`
**Created**: 2026-07-19
**Status**: Draft
**Input**: Operator provisioning portal that emits a **Medusa** `ProvisioningManifest` for `undevops` one-click client-shop deploy. Minimal scope: create/validate manifest, trigger operator pipeline.
**Owner**: `platform/undevops` + undrlla orchestration
**Pilot mode (decision)**: operator-only (recommended for MVP)
**Updated**: 2026-07-30 — client runtime = **Medusa** (`undreseller`), not Directus shop

## Summary

Provide a minimal, secure, and auditable **operator-first** provisioning portal to request a new **client marketplace** instance. The portal collects required inputs, validates them, and emits a `ProvisioningManifest` JSON (`manifest_version: 2.0-medusa`) that `undevops` consumes to provision **Medusa template-shop + Postgres + Redis + optional undrllanding storefront + TLS**.

Goal: reduce manual operator work and standardize provisioning inputs while preserving security-sensitive controls (secrets are not collected raw; use secret-env references or operator review step where required).

**Product context:** Undrlla both **creates/deploys** client marketplaces (Medusa via undreseller) and (on the flagship, deferred) **displays** them as a marketplace-of-marketplaces. This portal owns create/deploy manifests; the hub directory is a separate flagship feature. The manifest MUST carry hub opt-in fields (`list_on_hub`, `share_catalog_to_hub`) without re-collecting metadata later.

**Contract SoT (locked):** `ProvisioningManifest` + `contracts/medusa-provisioning-manifest.schema.json` is the **only** provision payload. undevops `003-undrlla-one-click-deploy` MUST ingest this schema (not a parallel thin DTO). See `ECOSYSTEM.md` Q7–Q9. **Do not** provision Directus as client commerce runtime.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Create a new site (Priority: P1)

An authorized operator fills the provisioning form, provides required metadata, requests a site, and receives a `manifest_id` and status. For the MVP the portal is operator-only; customer self-service is planned for later phases. The system validates input and either enqueues a provisioning job or flags for operator review if secret-scoped inputs are required.

**Why this priority**: This is the core value: enable provisioning with minimal operator involvement and standardized manifest output.

**Independent Test**: Submit a valid manifest via the portal UI or API and verify `undevops` receives a valid `ProvisioningManifest` in the expected format and status transitions to `queued`.

**Acceptance Scenarios**:

1. **Given** user is authenticated and authorized, **When** they submit required fields, **Then** system returns 202 Accepted with `manifest_id` and `next_steps: queued`.
2. **Given** manifest validation fails (missing fields or invalid tenancy), **When** user submits, **Then** system returns 4xx with field-level error messages.
3. **Given** manifest includes operator-only keys, **When** user submits, **Then** system marks manifest `requires_review: true` and creates an operator review ticket.

---

### Edge Cases

- What happens when requested domain is already taken? -> Respond with `conflict` and suggest alternatives.
- If the operator pipeline is down, manifests are saved and marked `retry_pending` with exponential backoff.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST accept a `ProvisioningManifest` via UI or API and validate it against the schema.
- **FR-002**: System MUST NOT collect plain-text secrets; for secrets, the portal MUST accept only `secret:<provider>/<key_env>` references. System MUST perform a pre-ingestion validation check with the target `undevops` secret store to confirm referenced secret keys exist before queuing the manifest.
- **FR-003**: System MUST return a deterministic `manifest_id` and initial `status` (queued/needs_review/invalid).
- **FR-004**: System MUST persist manifests and provide an append-only operation log and audit trail (`ManifestAuditLog`) for each manifest.
- **FR-005**: System MUST emit the validated `ProvisioningManifest` to the `undevops` ingestion endpoint when `status` == `queued`.
- **FR-006**: System MUST sanitize and validate domain names and DNS PTR constraints. System MUST acquire an atomic domain reservation lock during manifest validation to prevent concurrent provisioning race conditions on the same domain.
- **FR-007**: System MUST support operator review workflow for manifests flagged `requires_review`.
- **FR-008**: System MUST provide webhooks or polling endpoints for provisioning status updates. When the `undevops` ingestion endpoint is offline, manifests MUST transition to `retry_pending` with exponential backoff (up to 10 attempts max); if retries exhaust, manifest MUST route to a Dead-Letter Queue (DLQ) and trigger an operator alert.
- **FR-009**: System owner for the MVP is `platform/undevops`.
- **FR-010**: System MUST enforce RBAC permissions distinguishing `provisioning:request` (create manifest) from `provisioning:approve` (approve `requires_review` manifests). High-privilege provisions require `provisioning:approve`.
- **FR-011**: `ProvisioningManifest` MUST support hub fields for the federated marketplace hub (`001` FR-037): `list_on_hub` (boolean, default `true`), optional `hub_category_tags` (string array), optional `hub_short_description`, and `share_catalog_to_hub` (boolean, default `true` — when true, client catalog products/services are eligible for flagship search index sync & dual currency rendering). When either flag is true, successful provision MUST leave durable metadata (display_name, public domain/URL, tags, description, instance API base URL reference) for hub onboarding without re-collecting fields. Merchants MAY toggle these flags off anytime in 1 click via dashboard.
- **FR-012**: After a site is `ready`, operators MUST be able to **patch** hub flags (`list_on_hub`, `share_catalog_to_hub`, tags, description) without full re-provision; changes emit an audit log entry and optional undevops config sync hook.
- **FR-013**: Day-2 lifecycle (instance upgrade, suspend, destroy, restore) is **out of scope for `002`**; owned by undevops future specs. Portal MAY display status only.
- **FR-014**: Custom-domain manifests SHOULD require `provisioning:approve` (or equivalent) in MVP; rate-limit `provisioning:request` creates per actor.

### Key Entities

- **ProvisioningManifest**: JSON object with declarative provisioning fields (tenant_id, display_name, domain, region, billing_info_reference, addons, initial_admin_email, initial_content_seed, list_on_hub, hub_category_tags, hub_short_description).
- **ManifestAuditLog**: append-only audit records for changes and operator actions.

## ProvisioningManifest Schema (required)

### Canonical runtime (2026-07-29): Medusa client shops

**Source of truth for undevops ingest of client shops** is the Medusa-shaped manifest:

| Artifact | Path |
|----------|------|
| JSON Schema | [`contracts/medusa-provisioning-manifest.schema.json`](./contracts/medusa-provisioning-manifest.schema.json) |
| Example | [`contracts/medusa-manifest.example.json`](./contracts/medusa-manifest.example.json) |

Example (compact, **use this for new work**):

```json
{
  "manifest_version": "2.0-medusa",
  "tenant_id": "acme-tenant-001",
  "display_name": "Acme Store",
  "domain": "acme.undrlla.shop",
  "domain_mode": "platform_subdomain",
  "region": "eu-central-1",
  "runtime": {
    "engine": "medusa",
    "image": "ghcr.io/underundre/undreseller-template-shop:latest"
  },
  "services": {
    "postgres": true,
    "redis": true,
    "storefront_image": "ghcr.io/underundre/undrllanding:latest"
  },
  "billing_reference": "paddle:sub_01example",
  "initial_admin_email": "admin@acme.example",
  "addons": ["payments_paddle", "crypto_shkeeper"],
  "list_on_hub": true,
  "share_catalog_to_hub": true,
  "hub_category_tags": ["fashion", "eu"],
  "hub_short_description": "Acme streetwear storefront",
  "secrets": {
    "paddle_api_key_ref": "secret:paddle/api_key",
    "medusa_jwt_secret_ref": "secret:medusa/jwt_secret",
    "postgres_password_ref": "secret:postgres/password"
  },
  "env": {
    "TENANT_MODE": "client",
    "FEATURES": "shop"
  }
}
```

### Historical Directus-shaped example (do not use for client shops)

```json
{
  "manifest_version": "1.0",
  "tenant_id": "acme-tenant-001",
  "display_name": "Acme Store",
  "domain": "acme.undrlla.example",
  "region": "eu-central-1",
  "billing_reference": "stripe:acct_...",
  "initial_admin_email": "admin@acme.example",
  "addons": ["payments", "analytics"],
  "list_on_hub": true,
  "share_catalog_to_hub": true,
  "hub_category_tags": ["fashion", "eu"],
  "hub_short_description": "Acme streetwear storefront",
  "secrets": {
    "directus_service_token_ref": "secret:directus/service_token_env"
  }
}
```

Notes:
- `secrets.*` values MUST be secret references (format `secret:<provider>/<key>`), not raw secrets.
- `billing_reference` is an opaque reference (no card data collected). Prefer `paddle:…` pre–US LLC.
- `list_on_hub` / `share_catalog_to_hub` default **`true`** (008 economics: free hub traffic). Merchant may opt out post-ready via FR-012.
- `domain_mode`: `platform_subdomain` (free `*.undrlla.shop`) | `byo_custom` (merchant DNS) | `managed_purchase` (Undrlla registers at registrar cost).

## Design Decisions (defaults applied)

- **Primary user (MVP)**: operator-only (pilot). Customer self-service deferred.
- **Allowed `addons` for MVP**: `payments`, `analytics`, `sample-content`.
- **`billing_reference` format**: opaque external reference (e.g., `stripe:acct_...`).
- **Domain provisioning policy**: operator-managed for MVP (no automatic DNS provisioning).
- **Secrets handling**: accept only `secret:` references; raw secrets rejected.

## Success Criteria *(mandatory)*

- **SC-001**: Operators can provision a new site from a queued manifest in under **10 minutes** p95 warm-path (aligned with undevops 003 SC-001).
- **SC-002**: Manifest validation rejects invalid domains with 100% precision in tests.
- **SC-003**: At least 90% of manifests are `queued` (no review) for standard customers in the first 30 days of pilot.

## Acceptance Tests (brief)

- Submit a valid manifest via HTTP POST `/provisioning/manifests` (authenticated) and assert 202 + `manifest_id` and `status: queued`.
- Submit manifest with `secrets` containing raw values and assert 400 with explanation to use `secret:` references.
- Simulate `undevops` ingestion endpoint offline; ensure manifest saved with `status: retry_pending` and retries logged.

## Operator Review Flow (brief)

- Manifests with `requires_review: true` create an operator ticket with the audit trail and diff of requested configuration.
- Operator may `approve` or `reject` with a comment; approval transitions manifest to `queued` and triggers ingestion.

## Integrations / Outputs

- `undevops` ingestion API: POST `/api/provisioning/ingest` with JSON body `ProvisioningManifest`.
- Webhook subscription: `/provisioning/manifests/{id}/events` for status updates.

## Implementation Notes

- UI should reuse existing admin authentication and CSRF protections.
- Validation is performed server-side using JSON Schema for `ProvisioningManifest`.
- Do not store raw secrets; secret references are validated syntactically only.

## Checklist (quick)

 - [x] Spec file created in [repos/undrlla/specs/002-provisioning-portal/spec.md](repos/undrlla/specs/002-provisioning-portal/spec.md)
 - [x] ProvisioningManifest schema included
 - [x] Acceptance tests described
 - [x] Owner assigned (`platform/undevops`)


---

*Draft created by assistant.*
