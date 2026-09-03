# SpecKit Review: 002-provisioning-portal

**Reviewer**: antigravity
**Reviewed at**: 2026-07-21T10:37:11Z
**Commit**: ae4fbca508498ddcecfb8f73df3c100ee56103da (repos/undrlla)
**Artifacts reviewed**: spec.md. NOT FOUND (skipped): plan.md, tasks.md, data-model.md, research.md, quickstart.md, contracts/.

## Summary

Feature `002-provisioning-portal` is currently in draft specification stage (`spec.md` only). Core planning artifacts (`plan.md`, `tasks.md`, `data-model.md`, `contracts/`) are missing, which blocks the SpecKit implementation gate per Constitution Principle VI. The specification itself defines a solid high-level model for declarative site provisioning via `ProvisioningManifest`, but lacks domain lock synchronization, secret-reference verification guards, and failure recovery semantics.

## Findings

| ID | Severity | Area | Finding | Recommendation |
|---|---|---|---|---|
| F1 | CRITICAL | SpecKit Pipeline | Core pipeline artifacts `plan.md` and `tasks.md` are absent. Per Constitution Principle VI and Principle IX, `/speckit.implement` cannot proceed without verified plan and task definitions. | Complete the SpecKit pipeline stages (`/speckit.plan` and `/speckit.tasks`) before requesting implementation review. |
| F2 | CRITICAL | Security / Secret Handling | `spec.md:73-77, 89` requires `ProvisioningManifest` to accept `secret:<provider>/<key_env>` references without raw secrets. However, the system defines no validation protocol to verify that referenced secrets exist in the target `undevops` secret store prior to queuing the manifest. | Define a pre-ingestion secret reference validation check in `undevops` to prevent queuing broken manifests. |
| F3 | HIGH | Multi-Tenancy / Concurrency | `spec.md:36` specifies returning `conflict` on duplicate domains, but does not define an atomic domain reservation mechanism. Concurrent provisioning requests for the same custom domain can trigger race conditions. | Introduce a transient domain reservation lock during manifest validation and queuing. |
| F4 | HIGH | Reliability / Fault Tolerance | `spec.md:37, 99` specifies `retry_pending` status when the `undevops` ingestion pipeline is offline, but lacks retry caps, backoff parameters, dead-letter queue handling, or operator alerting. | Specify exponential backoff schedule, max retry limit (e.g. 10 attempts), and DLQ transition rules for unserviceable manifests. |
| F5 | HIGH | AuthZ / Access Control | `spec.md:113` notes UI should reuse existing admin authentication, but does not specify role-based authorization for operator review (`requires_review: true`). Any authenticated admin could approve sensitive infrastructure provisions. | Define explicit RBAC permissions (e.g., `provisioning:request` vs `provisioning:approve`) for manifest actions. |

## Alternative approaches considered

- **Synchronous Direct Provisioning**: Eliminating asynchronous queuing in favor of direct execution simplify state tracking, but risks browser timeouts and poor resilience against worker downtime. Asynchronous queue model in spec is preferred.

## VERDICT

```yaml
verdict: CRITICAL
reviewer: antigravity
reviewed_at: 2026-07-21T10:37:11Z
commit: ae4fbca508498ddcecfb8f73df3c100ee56103da
critical_count: 2
high_count: 3
medium_count: 0
low_count: 0
```
