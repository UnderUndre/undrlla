# Quickstart: undrlla-core (Tier-A) dev instance

**Spec**: `spec.md` | **Directus**: 12.x pinned v12.1.1

Spin up a single local `undrlla-core` marketplace instance for development. This mirrors what `undevops` does per client, minus the ACME/domain automation.

## Prerequisites

- Docker + Docker Compose
- Node.js 20+ (for building Directus extensions)
- A canonical settlement currency chosen for the instance (e.g. `EUR`)
- Provider sandbox creds (Stripe test, TON testnet, SHKeeper test) — injected as env, never committed (FR-035)

## 1. Configure environment

Copy the example env and fill secrets (kept out of source control):

```bash
cp .env.example .env
# set: DB creds, KEY/SECRET, PUBLIC_URL, canonical currency,
#      STRIPE_SECRET_KEY, STRIPE_WEBHOOK_SECRET,
#      TON_* , SHKEEPER_API_KEY, SHKEEPER_WEBHOOK_SECRET
```

## 2. Start Directus + Postgres

```bash
docker compose up -d db
docker compose up -d directus
```

## 3. Review and apply the schema snapshot

Review `schema.snapshot.yaml` before applying it. This quickstart targets a disposable local dev instance, but the same command shape is often copied into real environments, so schema changes stay review-first. For non-disposable targets, generate and inspect a diff through Directus' `/schema/diff` API before applying.

```bash
docker compose exec directus npx directus schema apply ./schema.snapshot.yaml
```

## 4. Build & load extensions

Payment/tax/notification extensions and the Tier-B extension-point seams (no flagship collections in `001`):

```bash
npm --prefix ./extensions ci
npm --prefix ./extensions run build
docker compose restart directus
```

## 5. Seed

Seeds the first Admin (credentials via env / out-of-band), a demo Tenant/vendor, sample products (physical + digital), a bookable service, and a delivery zone:

```bash
docker compose exec directus node ./scripts/seed.mjs
```

## 6. Run

- Directus API/admin: `http://localhost:8055`
- Storefront (`undrllanding`): `npm --prefix ./apps/storefront run dev` → `http://localhost:3000`
- Telegram Mini App: `npm --prefix ./apps/miniapp run dev` (point BotFather web app URL at the tunnel)

Both clients consume the same versioned contract in `contracts/` (FR-014).

## 7. Verify (smoke)

- REST + GraphQL serve products/orders/services (US1 AS1)
- No Tier-B flagship collections present (US1 AS2)
- Add multi-vendor cart → checkout splits into one Order + PaymentAttempt per vendor (`checkout_group_id`)
- Book a service slot → confirm → overlapping pending rejected
- Trigger a webhook twice → single state transition (idempotent)

## Verify secrets are injected, not baked

`schema.snapshot.yaml` and images MUST contain no provider secrets; confirm they resolve only from env/secret store (FR-035).

## Reset

```bash
docker compose down -v && docker compose up -d
```
