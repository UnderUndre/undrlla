# 004 — Identity handles & hub (one-pager / FR sketch)

**Status:** outline (not full SpecKit feature yet)  
**Date:** 2026-07-27  
**Depends on:** `ECOSYSTEM.md` (single IdP), `CONSTITUTION.md` §IV–§V  
**Dialog:** `.ai/dialogs/log/2026-07-27-grok.md`

## Intent

Global `@username` for the ecosystem + public hub page `undrlla.com/@user` as the “capital” surface that deep-links into undrlla modules (shop, housing, unet, undrepost, undevops projects, recruiting, matching).

## Locked product rules (constitution)

| Rule | Detail |
|------|--------|
| Scope | **Global unique** handle across all Undrlla apps |
| Hub | Canonical profile URL on flagship undrlla / undrllanding |
| Reclaim | Brands, countries, trademarks → verified lawful rights holders |
| Auth root | Not SMS/phone as sole recovery (prefer TOTP / WebAuthn) |
| Citizen | Badge on profile after 12 mo hybrid earn — signal, not handle gate |

## Suggested collections / fields (flagship Directus)

### `Handle` (or fields on `directus_users` + registry table)

| Field | Type | Notes |
|-------|------|--------|
| `username` | string unique, case-insensitive fold | Normalized: `[a-z0-9_]{3,32}` (TBD) |
| `user_id` | M2O user | Owner |
| `status` | `active` \| `reserved` \| `disputed` \| `transferred` | |
| `display_name` | string | Optional |
| `is_citizen` | computed/denorm | From citizenship status |
| `modules` | JSON | Which hub tiles to show |
| `claimed_at` | datetime | |
| `reserved_reason` | string nullable | system / iso / brand seed |

### `HandleClaim`

Trademark / country reclaim queue: evidence URLs, status `pending|approved|rejected`, forces rename of prior holder with free rename token.

### `HandleReservation` (seed)

Preload: `undrlla`, `unet`, `undevops`, `undrepost`, ISO2/ISO3 codes, obvious geo names — `enabled` for claim-only by verified orgs.

## FR sketch

- **FR-H01**: User MAY claim one free handle at signup or later if available.  
- **FR-H02**: Username unique globally (IdP authority = flagship).  
- **FR-H03**: Rename: cooldown (e.g. 30d) + optional history redirect.  
- **FR-H04**: Reserved list cannot be claimed by random users.  
- **FR-H05**: Verified claim flow transfers handle to rights holder; prior user gets rename prompt.  
- **FR-H06**: Public `GET /@/:username` hub returns profile + deep links (feature-flagged modules).  
- **FR-H07**: Apps (undrepost, unet portal) **display** same handle; they do not own a separate registry.  
- **FR-H08**: Citizenship badge visible when `undrlla.citizen`.  

## Non-goals (v1)

- Paid vanity auction (later).  
- Federated ActivityPub handles.  
- Per-tenant shop local usernames as global identity (shops may show display name only).

## Implementation home

| Option | When |
|--------|------|
| **A.** Fields on Directus users in flagship only | Fastest MVP |
| **B.** Small extension package `identity-handles` | Cleaner isolation |
| **C.** Full SpecKit `004-identity-handles/` | When scheduling implement |

**Recommend:** A for first ship; promote to B if claim disputes need workflow tables.

## Open

- Exact charset / unicode policy (homoglyphs).  
- Cookie domain / JWT still open in ECOSYSTEM.  
- Whether client-only Medusa/`undreseller` shops ever share handles (see `ECOSYSTEM` + undreseller note).
