---
name: Return an order and follow it to refund
description: Open a whole-order return against a completed Rye order and track it asynchronously to the shopper's refund.
api: openapi/rye-checkout-intents-openapi-original.yml
operations: [GetOrder, RequestReturn, GetReturn, CancelOrder]
---

# Return an order and follow it to refund

## Steps
1. **Confirm the order exists** — `GetOrder` (`GET /api/v1/orders/{id}`) to verify it is completed.
2. **Cancel instead, if unshipped** — if the order has not shipped, `CancelOrder` (`POST /api/v1/orders/{id}/cancel`) may be accepted; denials return a `CancellationDenialReasonCode` (e.g. `already_shipped`).
3. **Create the return** — `RequestReturn` (`POST /api/v1/returns`). Whole-order returns only; Rye enumerates the line items. Submitted for approval, then progresses asynchronously toward the refund.
4. **Track it** — poll `GetReturn` (`GET /api/v1/returns/{returnId}`) or listen for webhooks. Tenancy is scoped to the authenticated developer.

## Testing (staging)
Use the test-helper endpoints to drive a simulated return lifecycle: `CreateSimulatedReturn`, then `ApproveReturn` / `DenyReturn` / `RefundReturn` / `FailReturn`. See sandbox/rye-sandbox.yml.

## Rules
- Errors are JSON `{name, message, stack?}`; cancellation reasons/denials use the codes in errors/rye-decline-codes.yml.
- Returns are asynchronous - never assume the refund is immediate; drive off state or webhooks.
