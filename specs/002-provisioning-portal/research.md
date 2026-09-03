# Research & Design Decisions: 002-provisioning-portal

**Spec**: `spec.md` | **Owner**: `platform/undevops`

## D1 — Secret Handling via References

**Decision**: The portal accepts only opaque `secret:<provider>/<key_env>` references in `ProvisioningManifest.secrets` (FR-002). Raw secret values are strictly rejected.
**Pre-ingestion Validation**: Before queuing a manifest, the portal queries the `undevops` secret store API to verify that referenced keys exist in the target secret scope.

## D2 — Atomic Domain Reservation Locking

**Decision**: Domain collision prevention uses `DomainReservationLock` records with a 15-minute TTL during manifest validation.
**Rationale**: Eliminates race conditions when multiple operators or automated API requests submit manifests for identical custom subdomains concurrently.

## D3 — Queue Resilience and DLQ Routing

**Decision**: When the `undevops` ingestion endpoint is unreachable, manifests enter `retry_pending` status and retry using exponential backoff: `t = 2^attempt * 5s` up to 10 attempts.
**Failure Policy**: After 10 failed retries, status becomes `dlq` and an alert event is published to the operator notifications channel.

## D4 — Pilot Mode Operator-Only Scope

**Decision**: For v1 MVP, access is restricted to authorized operators (`platform/undevops`). Customer self-service provisioning UI and billing integration are deferred to v2.

## D4 — Hub listing opt-in on manifest (marketplace of marketplaces)

**Decision**: `ProvisioningManifest` carries `list_on_hub` (default false) plus optional tags/description (`002` FR-011). Provisioning still only creates/deploys; the flagship directory (`001` FR-037) consumes durable metadata later.

**Rationale**: Avoids re-onboarding when the hub ships; default-off protects clients who want private storefronts only.
