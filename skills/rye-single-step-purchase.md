---
name: One-shot purchase with Rye
description: Fire-and-forget checkout - create and trigger a purchase in a single call, then track fulfillment via webhooks.
api: openapi/rye-checkout-intents-openapi-original.yml
operations: [Purchase, GetCheckoutIntent, GetShipments, ListCheckoutIntents]
---

# One-shot purchase with Rye

Use when you already know exactly what to buy and want Rye to create the intent and start the order in one request.

## Steps
1. **Purchase** — `Purchase` (`POST /api/v1/checkout-intents/purchase`) with the product URL, `buyer`, quantity, `variantSelections`, `paymentMethod`, and price constraints. Rye creates the checkout intent and immediately triggers the purchase workflow.
2. **Track state** — subscribe to `checkout_intent.*` webhooks (HMAC-SHA256, `x-rye-signature`; verify with the SDK `events.unwrap` helper). Thin events carry only `source.id`; fetch full state with `GetCheckoutIntent` (`GET /api/v1/checkout-intents/{id}`).
3. **Follow shipments** — `GetShipments` (`GET /api/v1/checkout-intents/{id}/shipments`).
4. **Reconcile** — `ListCheckoutIntents` (`GET /api/v1/checkout-intents`) with cursor pagination (`after`/`before`/`limit`) and optional `state` filter.

## Rules
- Counts against the mutations/sec, checkout-intents/day, and concurrent-agents limits at once.
- On failure the intent lands in `failed` with `failureReason.code` (see errors/rye-decline-codes.yml).
- In staging no real order is placed and payment uses Stripe test cards (see sandbox/rye-sandbox.yml).
