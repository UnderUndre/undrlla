# Feature Specification: Provisioning Portal (002)

**Feature Branch**: `spec/002-provisioning-portal`
**Created**: 2026-07-19
**Status**: Draft
**Input**: User description: "Self-service provisioning portal to create new Directus instances (site provisioning) and emit a ProvisioningManifest consumable by `undevops` for operator-free provisioning. Minimal scope: create a manifest, validate it, and trigger operator pipeline."
**Owner**: `platform/undevops`
**Pilot mode (decision)**: operator-only (recommended for MVP)

## Summary

Provide a minimal, secure, and auditable self-service provisioning portal allowing operators or customers to request a new Directus-backed site instance. The portal collects required inputs, validates them, and emits a `ProvisioningManifest` JSON that the `undevops` operator pipeline consumes to provision infrastructure and Directus configuration.

Goal: reduce manual operator work and standardize provisioning inputs while preserving security-sensitive controls (secrets are not collected raw; use secret-env references or operator review step where required).

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
- **FR-002**: System MUST NOT collect plain-text secrets; for secrets, the portal must accept a `secret_reference` or instruct the user to complete an operator-only review step.
- **FR-003**: System MUST return a deterministic `manifest_id` and initial `status` (queued/needs_review/invalid).
- **FR-004**: System MUST persist manifests and provide an operation log and audit trail for each manifest.
- **FR-005**: System MUST emit the validated `ProvisioningManifest` to the `undevops` ingestion endpoint (see contract) when `status` == `queued`.
- **FR-006**: System MUST sanitize and validate domain names and DNS PTR constraints.
- **FR-007**: System MUST support operator review workflow for manifests flagged `requires_review`.
- **FR-008**: System MUST provide webhooks or polling endpoints for provisioning status updates.

- **FR-009**: System owner for the MVP is `platform/undevops`.

### Key Entities

- **ProvisioningManifest**: JSON object with declarative provisioning fields (tenant_id, display_name, domain, region, billing_info_reference, addons, initial_admin_email, initial_content_seed).
- **ManifestAuditLog**: append-only audit records for changes and operator actions.

## ProvisioningManifest Schema (required)

Example (compact):

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
  "secrets": {
    "directus_service_token_ref": "secret:directus/service_token_env"
  }
}
```

Notes:
- `secrets.*` values MUST be secret references (format `secret:<provider>/<key_env>`), not raw secrets.
- `billing_reference` is an opaque reference (no card data collected).

## Design Decisions (defaults applied)

- **Primary user (MVP)**: operator-only (pilot). Customer self-service deferred.
- **Allowed `addons` for MVP**: `payments`, `analytics`, `sample-content`.
- **`billing_reference` format**: opaque external reference (e.g., `stripe:acct_...`).
- **Domain provisioning policy**: operator-managed for MVP (no automatic DNS provisioning).
- **Secrets handling**: accept only `secret:` references; raw secrets rejected.

## Success Criteria *(mandatory)*

- **SC-001**: Operators can provision a new site from a queued manifest in under 10 minutes for standard templates.
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
