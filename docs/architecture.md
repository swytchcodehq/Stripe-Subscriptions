# Architecture

## Principles

1. **Swytchcode is the execution kernel.** Every Stripe API call is delegated to a Swytchcode method or workflow with a discovered canonical ID and contract.
2. **Services are independently deployable.** Each `services/*` package owns its own dependencies, build, and runtime entrypoint.
3. **Shared code is contract-only.** `shared/*` packages publish types, helpers, and adapters — never business logic specific to a service.
4. **Webhook truth wins.** Stripe webhooks are the source of truth for state transitions; service state is reconciled against them.

## Layers

```
            ┌────────────────────────────┐
            │  examples/ (express, ...)  │
            └─────────────┬──────────────┘
                          │
            ┌─────────────▼──────────────┐
            │       services/*           │
            │  (HTTP / queue handlers)   │
            └─────────────┬──────────────┘
                          │
            ┌─────────────▼──────────────┐
            │        shared/*            │
            │ (types, utils, storage,    │
            │  logger, test-helpers)     │
            └─────────────┬──────────────┘
                          │
            ┌─────────────▼──────────────┐
            │   Swytchcode runtime       │
            │   (compiled Stripe calls)  │
            └────────────────────────────┘
```

## Cross-cutting concerns

- **Idempotency** — every external mutation is keyed and persisted before issue.
- **Observability** — `shared/logger` enforces structured logs and trace context.
- **Storage** — `shared/storage` exposes a minimal port; each service picks its adapter.

See [service-index.md](service-index.md) for the per-service breakdown.
