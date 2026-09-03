# Tasks: 002-provisioning-portal

**Updated**: 2026-07-30 — Medusa-only; no Directus client-shop provision tasks.

## Tasks

1. [BE] P-T1 — Medusa ProvisioningManifest schema & collections
   - Owner: `backend-specialist`
   - Est: 2d
   - Deliverables: `ProvisioningManifest` (`manifest_version: 2.0-medusa`), `ManifestAuditLog`, `DomainReservationLock`; validate against `contracts/medusa-provisioning-manifest.schema.json` (not historical Directus 1.0 shape).

2. [BE] P-T2 — Secret reference validator & domain lock
   - Owner: `backend-specialist`
   - Est: 2d
   - Depends on: P-T1
   - Deliverables: Pre-ingestion secret reference checker against `undevops` secret store (Paddle keys, Medusa secrets as refs only), atomic domain lock handler.

3. [BE] P-T3 — Operator review & RBAC permissions
   - Owner: `backend-specialist`
   - Est: 2d
   - Depends on: P-T1
   - Deliverables: `requires_review` workflow, `provisioning:request` vs `provisioning:approve` RBAC guards, review comments audit log. MVP = operator-only.

4. [BE] P-T4 — undevops ingest client, retry worker & DLQ
   - Owner: `devops-engineer`
   - Est: 2d
   - Depends on: P-T2
   - Deliverables: HMAC-signed POST of manifest to undevops `003` ingest; status poll/webhook; exponential backoff (max 10); DLQ + operator alert. Targets **Medusa + storefront** job, not Directus shop snapshot.

5. [TEST] P-T5 — Contract tests & operator runbook
   - Owner: `test-engineer`
   - Est: 2d
   - Depends on: P-T2, P-T3, P-T4
   - Deliverables: OpenAPI contract tests for `/provisioning/manifests`; fixture = `contracts/medusa-manifest.example.json`; error scenarios; Gate 0 operator runbook (DNS, secrets, Medusa `/health`).

## Dependency Graph

```
P-T1 → P-T2 → P-T4
P-T1 → P-T3
P-T2 + P-T3 + P-T4 → P-T5
```

## Parallel Lanes

| Lane | Tasks |
|------|-------|
| A Schema + secrets | P-T1 → P-T2 → P-T4 |
| B Review RBAC | P-T1 → P-T3 |
| C Tests | P-T5 after A+B |

## Agent Summary

| Tag | Tasks |
|-----|-------|
| [BE] | P-T1, P-T2, P-T3 |
| [OPS]/devops | P-T4 |
| [TEST] | P-T5 |

## Total estimate: 10 dev-days
