---
name: Refund an EBT SNAP payment
description: Issue a full or partial refund of a captured EBT Payment and confirm the refund succeeded via receipt data.
api: https://docs.joinforage.app/reference/payment-refunds
method: generated
source: https://docs.joinforage.app/reference/create-payment-refund
operations:
- POST /payments/{payment_ref}/refunds/
- GET /payments/{payment_ref}/refunds/{refund_ref}/
- GET /payments/{payment_ref}/refunds/
---

# Refund an EBT SNAP payment

Refund SNAP or EBT Cash benefits to the originally charged PaymentMethod.

## Steps
1. **Create the refund** — `POST /payments/{payment_ref}/refunds/` with the `amount` to refund. Server-side only. Use Forage version `2024-01-08`+ so the response includes populated `receipt` data. The endpoint always returns `201` (a `PaymentRefund` record is created even on failure).
2. **Confirm success** — check that `status` is `succeeded`. If it is not yet resolved, `GET /payments/{payment_ref}/refunds/{refund_ref}/` periodically. On `failed`, inspect `receipt.message` for the reason.
3. **List refunds** — `GET /payments/{payment_ref}/refunds/` to see all refunds for a payment.

## Rules
- Server-side only; include an `Idempotency-Key`.
- Prefer refunds over voids for EBT (better paper trail); voids apply to POS Terminal / HSA-FSA only.
- Listen for the `REFUND_STATUS_UPDATED` webhook instead of polling where possible (`asyncapi/forage-webhooks.yml`).
- Refund declines surface as EBT codes (e.g. `ebt_error_61` return exceeds authorization, `ebt_error_62` restricted/frozen card) — see `errors/forage-decline-codes.yml`.
