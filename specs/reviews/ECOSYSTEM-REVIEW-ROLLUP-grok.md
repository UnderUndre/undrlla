# SpecKit Review rollup: Undrlla ecosystem (grok)

**Reviewed at**: 2026-07-23T10:06:54Z
**Reviewer**: grok

| Repo / feature | Verdict | Top issue |
|---|---|---|
| undrlla/001-init-from-directus | HIGH | Vendor invite vs self-signup; TG auth vs Mini App |
| undrlla/002-provisioning-portal | CRITICAL | Manifest ≠ undevops 003 payload |
| undrlla/003-housing-donations | HIGH | Sub entity + chargeback accounting; stale gate |
| undevops/001-init | HIGH | MCP token + AI secret redaction |
| undevops/002-telegram-control | CRITICAL | Missing plan/tasks; bot auth |
| undevops/003-undrlla-one-click | CRITICAL | Contract dualism + missing artifacts |
| undrllanding/001-storefront | CRITICAL | Spec/code gap; TG auth conflict; scope |
| undrepost/001-init-from-payload | CRITICAL | Obsolete IdP (Medusa/Better-Auth vs Directus) |
| unet/001-init-netstack | HIGH | No paid control plane in undrlla |
| undesign | MEDIUM (advisory) | Adoption only |

## Implement gate note
Constitution Principle VI lists reviewers: claude, codex, antigravity, gemini, copilot. **`grok` may not count** toward the 2-external-reviewer gate until constitution is amended. Re-run reviews with listed providers for shipping gates.

## Suggested fix order
1. undevops 003 ↔ undrlla 002 manifest SoT
2. undrllanding Wave-1 + TG auth decision
3. undrepost IdP rewrite to Directus
4. undrlla 003 subscription data-model + tasks refresh
5. unet billing FR in undrlla

## Fixes applied 2026-07-23
All items above addressed in-repo specs (see ECOSYSTEM.md “SpecKit review fixes applied”). Re-run listed-provider `/speckit.review` for implement gate; grok reviews remain advisory for Principle VI provider list.
