# Stripe Subscriptions

A monorepo of production-ready Stripe Subscriptions services built on top of [Swytchcode](https://swytchcode.com).

Each service in `services/` is an independently deployable unit that delegates Stripe API execution to the Swytchcode runtime, ensuring contract-validated, version-pinned integration calls.

## Layout

- `services/` — independently deployable subscription services
- `shared/` — cross-service code (types, utils, storage, logger, test helpers)
- `examples/` — reference integrations (Express, Next.js, Hono)
- `docs/` — architecture, contribution guide, local testing, service index

## Getting started

```bash
pnpm install
cp .env.example .env
# fill in Stripe + Swytchcode credentials
```

See [docs/architecture.md](docs/architecture.md) for an overview and [docs/service-index.md](docs/service-index.md) for the service catalog.

## Services

| Service | Status | Purpose |
|---|---|---|
| [`activation-service`](services/activation-service) | implemented | Convert sign-ups into active subscriptions — never grants access from a redirect alone, only after verified `invoice.paid` |
| [`first-payment-recovery-service`](services/first-payment-recovery-service) | implemented | Recover failed initial charges — declines, SCA / 3DS, async PMs, and `incomplete_expired` |
| [`renewal-recovery-service`](services/renewal-recovery-service) | implemented | Dunning + recovery for renewals — `past_due` grace, `unpaid` lockout, `restored` on recovery |
| `plan-change-service` | scaffold | Upgrades, downgrades, proration |
| `customer-portal-service` | scaffold | Self-serve customer portal flows |
| `entitlement-sync-service` | scaffold | Keep app-side entitlements in sync with Stripe state |
| `webhook-reliability-service` | scaffold | Idempotent, replayable webhook handling |
