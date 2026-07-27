---
name: Run a Fully Hosted EBT checkout
description: Create a Fully Hosted Forage checkout session, redirect the customer to complete PIN entry, and retrieve the resulting Order outcome.
api: https://docs.joinforage.app/docs/fully-hosted
method: generated
source: https://docs.joinforage.app/reference/create-session
operations:
- POST /o/token/
- POST /sessions/
- GET /orders/{order_ref}/
- POST /orders/{order_ref}/refund_all/
---

# Run a Fully Hosted EBT checkout

The lowest-code integration: Forage hosts the entire EBT checkout UI.

## Steps
1. **Mint a server auth token** — `POST /o/token/` with Client ID + Client Secret.
2. **Create the session** — `POST /sessions/` with the cart `product_list` (pass each item's `tax_rate` so Forage computes taxes) and `customer_id`. Include an `Idempotency-Key`. This creates an `Order`; store the returned `ref` (the `order_ref`) and `redirect_url`.
3. **Redirect the customer** — point them at `redirect_url` to enter their EBT PIN in the Forage-hosted UI. On completion Forage returns them to your `success_redirect_url` (or `cancel_redirect_url`).
4. **Retrieve the outcome** — `GET /orders/{order_ref}/` to read the transaction result, `sales_tax_applied`, and per-item tender breakdown.
5. **(Optional) Refund** — `POST /orders/{order_ref}/refund_all/` to refund the whole order, or use the by-product / partial refund endpoints.

## Rules
- Fully Hosted and Custom integrations are the only ones that create `Orders`; SDK integrations use `Payments` instead.
- Listen for `ORDER_STATUS_UPDATED` and `REFUND_STATUS_UPDATED` webhooks (`asyncapi/forage-webhooks.yml`).
- Auth, idempotency, error, and rate-limit rules per `conventions/forage-conventions.yml`.
