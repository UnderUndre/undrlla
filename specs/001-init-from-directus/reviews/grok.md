# SpecKit Review: 001-init-from-directus

**Reviewer**: grok
**Reviewed at**: '"$ts"'
**Commit**: '"$c001"'
**Artifacts reviewed**: spec.md, plan.md, data-model.md, research.md, contracts/openapi.yaml, quickstart.md; cross-checked ECOSYSTEM.md, FLOWS-AUDIT.md, constitution Principle VI

## Summary

Tier-A marketplace design is unusually deep (payments, multi-vendor cart, tax, PII, webhooks). Headline weakness: **scope sprawl and ecosystem drift** — deferred Tier-B FRs (hub MoR, logistics, MAQ, unet) are partially product-locked outside this feature while vendor onboarding remains invite-only, conflicting with later self-signup workforce decisions. Missing plan/tasks completeness relative to FR surface area is less critical than **identity and client-boundary ambiguity** for implementers.

## Findings

| ID | Severity | Area | Finding | Recommendation |
|---|---|---|---|---|
| F1 | HIGH | Stakeholder / consistency | FR-025 locks **admin-invited vendors only**; ECOSYSTEM + FR-038/039 lock **self-signup** workers/housing pros. Spec never reconciles when self-signup applies (flagship only? which roles?). | Add Clarification: Tier-A client = invite-only vendors; flagship workforce roles = self-signup + KYC gate; table of roles. |
| F2 | HIGH | Cross-ecosystem | FR-014/FR-017: Telegram Mini App ships in `001`, but **Telegram-based authentication is deferred**. `undrllanding` spec requires `/auth/telegram`. Contradiction blocks storefront implement. | Either promote TG auth into `001` MVP or demote Mini App auth stories to "password/JWT only until flagship auth ship". Align undrllanding. |
| F3 | HIGH | Hidden assumption | FR-037 Hub **MoR** assumes flagship can be seller-of-record (tax, chargebacks, possibly licensing). No legal/ops gate, no "catalog-only until MoR ready" slice in this feature. | Split FR-037 into FR-037a catalog/search and FR-037b MoR checkout; MoR behind compliance checklist. |
| F4 | MEDIUM | Failure modes | Multi-provider crypto (`direct_to_vendor` vs collect-and-forward) documented, but **partial multi-leg hub failure UX** and customer communication for mixed outcomes only sketch FR-024 spirit. | Specify customer-visible states and notification events for `partially_paid` / `partially_failed` checkout groups. |
| F5 | MEDIUM | Security | Guest `checkout_secret` and `anon_token` are strong (FR-034); less clear how **cross-instance hub revalidation credentials** work (service account per client?). | When hub lands, require mTLS or signed service tokens per instance; document in security section. |
| F6 | MEDIUM | Edge case | FR-021 vendor delivery zones **and** FR-038 Undrlla courier coexist; checkout selection / double-charge not specified in `001`. | Add edge case: mutually exclusive delivery modes; one fee path. |
| F7 | LOW | Stakeholder | "Passport" appears in product narrative outside this file; `001` still says Mutual Aid for "passport or housing" in US3 without linking to `003` package language. | Cross-link `003` housing+CBI as the concrete passport path. |
| F8 | CRITICAL | Artifacts / gate | Constitution VI requires analyze + 2 external reviews before implement. This review is **grok** (not in listed provider set: claude/codex/antigravity/gemini/copilot). Gate tooling may **not count** this file. | Re-run `/speckit.review` with a listed provider, or amend constitution to include `grok`. Treat this as advisory until then. |

## Alternative approaches considered

- **Handoff checkout** instead of Hub MoR for v1 federation (lower legal load; worse UX) — still valid if MoR blocked.
- **Shared auth service** instead of Directus-as-IdP — rejected in ECOSYSTEM for MVP; flag if Directus becomes bottleneck.

## VERDICT

```yaml
verdict: HIGH
reviewer: grok
reviewed_at: '"$ts"'
commit: '"$c001"'
critical_count: 1
high_count: 3
medium_count: 3
low_count: 1
```

**Note**: CRITICAL is process/gate (provider tag), not a hole in marketplace money flows. Product HIGH findings should be clarified before large implement waves.
