# SpecKit Review: 001-init-from-directus

**Reviewer**: copilot
**Reviewed at**: 2026-07-19T02:15:43Z
**Commit**: 29ff0d00932b20ade5f1809ef2a964c115f18c04
**Artifacts reviewed**: spec.md, plan.md, tasks.md, data-model.md, research.md, quickstart.md, contracts/openapi.yaml, .specify/memory/constitution.md

## Summary

The current artifacts are much stronger than the stale `claude` review: the data model, research note, quickstart, and contract now exist, and the Tier-B scope boundary is mostly cleaned up. The main remaining weakness is that the published API/test surface is still thinner than the spec's required marketplace workflows, and one quickstart command bypasses the repo's safety posture around schema application.

## Findings

| ID | Severity | Area | Finding | Recommendation |
| -- | -------- | ---- | ------- | -------------- |
| F1 | HIGH | Process / safety | `quickstart.md:33` tells operators to run `directus schema apply --yes`. That bypasses interactive confirmation for a schema-changing operation, conflicts with the attached coding standing order against `--yes`/bypass flags, and creates an avoidable destructive-change path if copied outside a disposable dev database. | Replace the command with a review-first flow: generate/inspect the Directus schema diff or snapshot plan, then apply interactively after explicit operator confirmation. If a dev-only fast path is retained, label it disposable-local-only and avoid `--yes`. |
| F2 | HIGH | Contract / traceability | `contracts/openapi.yaml:7` explicitly says the shared storefront/Mini App contract is "not exhaustive", but `tasks.md:74-78` makes contract tests and e2e validation the gate for the core. The contract currently covers catalog/cart/basic checkout/bookings/refund-request creation, but not notification delivery, vendor onboarding/payout setup, digital entitlement/download, tax quote/VIES input, refund approval/provider confirmation, or booking payment-policy behavior required by `spec.md:195-199` and `spec.md:246-250`. | Either expand the versioned contract to cover every client-visible workflow in SC-012 through SC-016, or explicitly split generated Directus REST/GraphQL surfaces from custom `/store/*` endpoints and add contract tests for both. |
| F3 | HIGH | AuthZ / workflow | `contracts/openapi.yaml:130-153` exposes one booking transition endpoint whose `action` enum includes `confirm`, `reject`, `cancel`, and `reschedule`, while `spec.md:188` says only the provider confirms/rejects and either party can cancel/reschedule. The contract does not encode actor-specific permissions, so implementation can easily ship a customer-confirm/reject privilege bug. | Define allowed transitions per actor/role in the contract and tests. Consider separate provider/customer transition endpoints or an explicit transition matrix enforced by Directus policy plus server-side guards. |
| F4 | MEDIUM | Payments / settlement | `spec.md:194` requires crypto adapters to make explicit whether payment is direct-to-vendor or collect-and-forward, and how the platform fee is settled. `tasks.md:28` only names quotes, wallet address, HMAC verification, and mismatch handling; `research.md:21-27` chooses a common abstraction but does not pick the crypto settlement model. | Add a concrete v1 settlement decision per TON/SHKeeper adapter, including fee collection, refund source, reconciliation owner, and what happens when a vendor changes payout wallets mid-order. |
| F5 | MEDIUM | Edge case | `spec.md:193` says a multi-vendor checkout creates independent vendor orders/payment attempts, but neither the contract nor tasks define user-visible semantics when one vendor order succeeds and another fails, expires, or is refunded in the same `checkout_group_id`. | Specify partial checkout outcomes: whether successful vendor orders remain committed, how failed vendor orders are surfaced, whether inventory holds are released per vendor, and how the storefront/Mini App present mixed states. |
| F6 | MEDIUM | Privacy / data model | The spec requires delivery addresses, guest contact handles, and VIES VAT validation (`spec.md:195-198`), while `contracts/openapi.yaml:92-95` accepts `contact_handle` and `delivery_address` as unstructured strings/objects. The data model names these fields but does not define retention, masking/FLS, validation, or log-redaction for address/VAT/contact PII. | Add PII handling rules to `data-model.md` and U-T14: structured address/VAT fields, retention/deletion policy, FLS masks, audit-log redaction, and tests that vendor/customer roles cannot over-read PII. |
| F7 | LOW | Constitution alignment | `plan.md:4` states the root constitution is a domain mismatch and that nothing in the plan claims conformance. That may be true for marketplace domain rules, but this review is still being run through the root SpecKit gate, so governance applicability is ambiguous rather than explicitly scoped. | Add a short governance note to the feature directory clarifying which root SpecKit principles apply to nested `repos/undrlla` artifacts, and whether a repo-local constitution will replace only domain rules or also pipeline rules. |

## Alternative approaches considered

- **API contract strategy**: the project can either rely on Directus' generated OpenAPI/GraphQL as the exhaustive contract, or maintain a curated storefront contract for `/store/*`. The current artifacts point to both; choosing one as authoritative would reduce contract-test ambiguity.
- **Crypto settlement**: direct-to-vendor wallet is simpler operationally but makes platform-fee capture and refunds harder; collect-and-forward centralizes reconciliation but increases custody/compliance surface. The spec requires this decision but has not made it yet.

## VERDICT

```yaml
verdict: HIGH
reviewer: copilot
reviewed_at: 2026-07-19T02:15:43Z
commit: 29ff0d00932b20ade5f1809ef2a964c115f18c04
critical_count: 0
high_count: 3
medium_count: 3
low_count: 1
```
