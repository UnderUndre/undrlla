# SpecKit Review: 002-provisioning-portal

**Reviewer**: grok
**Reviewed at**: '"$ts"'
**Commit**: '"$c001"'
**Artifacts reviewed**: spec.md, plan.md, tasks.md, data-model.md, research.md, contracts/; cross-checked undevops/003, ECOSYSTEM.md

## Summary

Clean minimal portal idea (manifest in → undevops). Strong: secret-ref pattern, domain lock, RBAC split request/approve. Weak: **contract dualism with undevops 003**, incomplete day-2 lifecycle, and hub fields without federation readiness criteria.

## Findings

| ID | Severity | Area | Finding | Recommendation |
|---|---|---|---|---|
| F1 | CRITICAL | Cross-repo consistency | ECOSYSTEM locks **this manifest as SoT**; undevops `003-undrlla-one-click-deploy` defines a **different** payload (`POST /deploy/marketplace`, HMAC, undesign_theme). Implementers have two truths. | Rewrite undevops 003 to **ingest ProvisioningManifest**; deprecate alternate payload or map 1:1. |
| F2 | HIGH | Failure modes | Retry/DLQ to undevops ingestion specified; **no restore/suspend/destroy/upgrade** tenant flows. | Add FR stubs or explicit out-of-scope for day-2 ops with owner undevops. |
| F3 | HIGH | Security | Operator-only MVP OK; customer self-service later. Manifest may hold **domain + admin email + hub flags** — abuse if request role leaks. | Rate-limit + audit every manifest create; require approve for custom domains. |
| F4 | MEDIUM | Hidden assumption | Assumes undevops always reachable enough for secret pre-check; offline undevops → stuck? | Document fail-closed vs queue-without-secret-check policy. |
| F5 | MEDIUM | Edge case | `list_on_hub` / `share_catalog_to_hub` set at provision; no flow to **toggle later** without re-provision. | FR: patch hub flags post-deploy via admin API. |
| F6 | LOW | Stakeholder | "under 10 minutes" SC-001 vs undevops "60 seconds" — mismatched SLOs. | Align success criteria across repos. |

## Alternative approaches considered

- GitOps declarative desired-state for tenants instead of imperative manifest queue.
- Direct undevops UI without undrlla portal (reject if undrlla owns commercial onboarding).

## VERDICT

```yaml
verdict: CRITICAL
reviewer: grok
reviewed_at: '"$ts"'
commit: '"$c001"'
critical_count: 1
high_count: 2
medium_count: 2
low_count: 1
```
