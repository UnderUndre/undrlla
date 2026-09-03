# Data Model: 002-provisioning-portal (undevops)

**Spec**: `spec.md` | **Owner**: `platform/undevops`

This data model defines the self-service provisioning portal entities for Directus instance provisioning via `undevops`.

## Collections

### ProvisioningManifest

declarative site provisioning payload.

- `id` (uuid, PK)
- `manifest_version` (string, e.g. "1.0")
- `tenant_id` (string, unique slug, e.g. "acme-store-001")
- `display_name` (string)
- `domain` (string, unique, domain-sanitized)
- `region` (string, e.g. "eu-central-1")
- `billing_reference` (string, opaque e.g. "stripe:acct_...")
- `initial_admin_email` (string)
- `addons` (json array, e.g. `["payments", "analytics"]`)
- `secrets` (json object, string map of `secret:<provider>/<key_env>` references)
- `status` (enum: `draft` | `queued` | `requires_review` | `in_progress` | `completed` | `failed` | `retry_pending` | `dlq`)
- `requires_review` (boolean, default false)
- `created_by` (M2O Directus user)
- `approved_by` (M2O Directus user, nullable)
- `created_at` (datetime)
- `updated_at` (datetime)

### ManifestAuditLog

Append-only audit trail for manifest state transitions and operator actions.

- `id` (uuid, PK)
- `manifest_id` (M2O ProvisioningManifest)
- `actor_id` (M2O Directus user)
- `action` (enum: `created` | `validated` | `queued` | `flagged_review` | `approved` | `rejected` | `ingested` | `failed` | `retry_attempt` | `routed_dlq`)
- `previous_status` (string, nullable)
- `new_status` (string)
- `details` (json object, e.g. validation errors or review comments)
- `created_at` (datetime)

### DomainReservationLock

Transient lock table for domain reservation during manifest validation and queuing.

- `domain` (string, PK)
- `manifest_id` (string)
- `locked_at` (datetime)
- `expires_at` (datetime)

## Roles & Permissions

- `provisioning:request` permission: allows creating and submitting manifests.
- `provisioning:approve` permission: required for operators to review, approve, or reject manifests marked `requires_review: true`.
- Audit logs are read-only for operators and append-only for system services.
