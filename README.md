# Stripe Subscriptions

Production-ready Stripe subscription services built on [Swytchcode](https://swytchcode.com). Each service handles one subscription lifecycle problem and delegates all Stripe API calls to the Swytchcode runtime.

## Setup

```bash
# 1. Install dependencies
npm install

# 2. Copy and fill environment variables (see table above)
cp .env.example examples/express/.env

# 3. Install swytchcode
Install CLI https://cli.swytchcode.com/

# 4. Initiate swytchcode
swytchcode bootstrap
```



## Run the demo

```bash
cd examples/express
npm run dev
```

Server starts at `http://localhost:3000`.

## Services

| Service | Status | Purpose |
|---|---|---|
| [`activation-service`](services/activation-service) | ready | Convert sign-ups to active subscriptions — access granted only after verified `invoice.paid` |
| [`first-payment-recovery-service`](services/first-payment-recovery-service) | ready | Recover failed initial charges — declines, SCA/3DS, async payment methods |
| [`renewal-recovery-service`](services/renewal-recovery-service) | ready | Dunning for renewals — `past_due` grace, `unpaid` lockout, `restored` on recovery |
| `plan-change-service` | scaffold | Upgrades, downgrades, proration |
| `customer-portal-service` | scaffold | Self-serve billing portal flows |
| `entitlement-sync-service` | scaffold | Keep app entitlements in sync with Stripe |
| `webhook-reliability-service` | scaffold | Idempotent, replayable webhook handling |

## Prerequisites

- Node.js ≥ 20
- npm ≥ 10
- [Stripe CLI](https://docs.stripe.com/stripe-cli) — for local webhook forwarding
- [Swytchcode CLI](https://swytchcode.com) — for runtime integration setup

## Environment variables

Copy the template and fill in your credentials before starting the server.

```bash
cp .env.example examples/express/.env
```

| Variable | Required | Where to get it |
|---|---|---|
| `STRIPE_SECRET_KEY` | Yes | [Stripe Dashboard → Developers → API keys](https://dashboard.stripe.com/test/apikeys) — use the `sk_test_...` key for local dev |
| `STRIPE_PUBLISHABLE_KEY` | No | Same page as above — `pk_test_...` (only needed if your frontend calls Stripe.js directly) |
| `STRIPE_WEBHOOK_SECRET` | Yes | Run `stripe listen --forward-to localhost:3000/webhooks/stripe` — the CLI prints a `whsec_...` value; paste it here |
| `SWYTCHCODE_API_KEY` | Yes | [Swytchcode Dashboard](https://swytchcode.com) → Settings → API keys |
| `PORT` | No | Defaults to `3000` |
| `NODE_ENV` | No | `development` locally, `production` in deployments |
| `LOG_LEVEL` | No | `info` (options: `debug`, `info`, `warn`, `error`) |
| `DATABASE_URL` | No | Your Postgres connection string — only required when using a persistent store instead of the in-memory default |
| `REDIS_URL` | No | Your Redis connection string — only required for webhook deduplication in `webhook-reliability-service` |



### Endpoints

| Method | Path | Description |
|---|---|---|
| `GET` | `/healthz` | Health check |
| `POST` | `/checkout` | Create a Stripe Checkout session |
| `GET` | `/activation/:accountId` | Get current activation state for an account |
| `POST` | `/webhooks/stripe` | Stripe webhook receiver |
| `POST` | `/plan-change/preview` | Preview a plan change with proration |
| `POST` | `/plan-change/apply` | Apply a plan change |

## Test with Stripe CLI

**1. Forward webhooks to your local server:**

```bash
stripe listen --forward-to localhost:3000/webhooks/stripe
```

Copy the printed `whsec_...` value into `.env` as `STRIPE_WEBHOOK_SECRET`, then restart the server.

**2. Trigger test events:**

```bash
# Subscription activation flow
stripe trigger checkout.session.completed
stripe trigger invoice.paid

# Renewal recovery flow
stripe trigger invoice.payment_failed

# Plan change flow
stripe trigger customer.subscription.updated
```

**3. Check activation state:**

```bash
curl http://localhost:3000/activation/<accountId>
```

## Repo layout

```
services/   — subscription service modules (one problem per file)
shared/     — cross-service types, utils, logger, storage, test helpers
examples/   — reference integrations (Express, Next.js, Hono)
docs/       — architecture, service index, local testing guide
```
