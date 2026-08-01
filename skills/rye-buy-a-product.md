---
name: Buy any product with Rye
description: Turn a product URL into a completed order using Rye's Universal Checkout API - look up the product, create a checkout intent, then confirm with a payment method.
api: openapi/rye-checkout-intents-openapi-original.yml
operations: [Lookup, CreateCheckoutIntent, GetCheckoutIntent, ConfirmCheckoutIntent, GetCheckoutIntentOrder]
---

# Buy any product with Rye

Complete a purchase on any supported merchant from a product URL, without leaving your app.

## Auth & environment
- Send your API key in the `Authorization` header. Staging and production use **separate keys** and are fully isolated.
- Start in staging: `https://staging.api.rye.com/api/v1/` (no real orders; Stripe test cards). Go live at `https://api.rye.com/api/v1/`.

## Steps
1. **(Optional) Look up the product** — `Lookup` (`GET /api/v1/products/lookup?url=...`) to preview price/variants before buying.
2. **Create the checkout intent** — `CreateCheckoutIntent` (`POST /api/v1/checkout-intents`) with the product URL, `buyer` (note: `buyer.country` must be UPPERCASE; phone format `010-010-0101`), quantity, and any `variantSelections`. Optionally set `maxTotalPrice` / `maxShippingCost` constraints.
3. **Poll for the offer** — `GetCheckoutIntent` (`GET /api/v1/checkout-intents/{id}`) until state is `awaiting_confirmation`. The `offer.cost` breakdown (subtotal/tax/shipping, all in `amountSubunits`) is now populated. Prefer webhooks (`checkout_intent.*`) over tight polling.
4. **Confirm with payment** — `ConfirmCheckoutIntent` (`POST /api/v1/checkout-intents/{id}/confirm`) with a tokenized `paymentMethod` (Stripe token, Basis Theory, drawdown, or x402). Confirming before the intent reaches `awaiting_confirmation` returns 404 — poll first.
5. **Retrieve the order** — `GetCheckoutIntentOrder` (`GET /api/v1/checkout-intents/{id}/order`) once state is `completed`.

## Rules
- **Money is in minor units**: `{ currencyCode, amountSubunits }` (e.g. $15.00 -> 1500).
- **No Idempotency-Key**: the checkout intent *is* the idempotency unit. Re-confirming a placed intent returns 409 - do not retry blindly.
- **Errors**: JSON `{name, message, stack?}`. On `failed` state, read `failureReason.code` (see errors/rye-decline-codes.yml) - e.g. `product_out_of_stock`, `constraint_total_price_exceeded`.
- **Rate limits**: 5 mutations/sec, 50 checkout intents/day, 10 concurrent agents. Honor `RateLimit` headers; back off on 429.
