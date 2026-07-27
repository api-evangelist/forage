---
name: Accept an EBT SNAP payment (SDK / Custom)
description: Tokenize a customer's EBT card, check its balance, create a Payment, and capture it using the Forage Payments API and Forage Elements.
api: https://docs.joinforage.app/reference/introduction
method: generated
source: https://docs.joinforage.app/reference/introduction
operations:
- POST /o/token/
- POST /session_token/
- POST /payment_methods/
- GET /payment_methods/{payment_method_ref}/
- POST /payments/
- POST /payments/{payment_ref}/capture_payment/
---

# Accept an EBT SNAP payment

Use this flow to charge a customer's EBT SNAP (or EBT Cash) benefits online.

## Prerequisites
- A Forage account with an app registered in the dashboard (Client ID + Client Secret).
- Forage Elements mounted client-side (`forage-js-sdk`, iOS `ForageSDK`, or Android `com.joinforage:forage-android`) to collect card + PIN. Raw PANs/PINs never touch your server.

## Steps
1. **Mint a server auth token** — `POST /o/token/` with Client ID + Client Secret. Store the `access_token` (long-lived, server-side only, default 7-day expiry).
2. **Mint a session token** — `POST /session_token/` using the auth token. It expires in 15 minutes and is passed to client-side SDK methods.
3. **Tokenize the EBT card** — client mounts the Forage card Element and calls the SDK tokenize method, or you `POST /payment_methods/` with the session token. Pass `customer_id` (strongly recommended). Store the returned `ref`.
4. **(Optional) Balance check** — for Custom flows, create a balance session and, after PIN entry, `GET /payment_methods/{payment_method_ref}/` and read `balance`. FNS prohibits balance inquiries when guest checkout is offered.
5. **Create the Payment** — `POST /payments/` with the `payment_method` ref and amount. Server-side only. Include an `Idempotency-Key` (UUID). Payments expire 30 minutes after creation.
6. **Capture** — collect the PIN via the Forage PIN Element and call the SDK `capturePayment()`, or defer to the server with `POST /payments/{payment_ref}/capture_payment/`. On success the response includes `receipt` data.

## Rules
- Auth: OAuth 2.0 bearer tokens in the `Authorization` header (see `authentication/forage-authentication.yml`).
- Idempotency: send `Idempotency-Key` on all create POSTs; keys are retained 24h (`conventions/forage-conventions.yml`).
- Errors: handle EBT decline codes (`ebt_error_51` insufficient funds, `ebt_error_55` invalid PIN, `ebt_error_75` PIN tries exceeded) from `errors/forage-decline-codes.yml`; API errors follow the envelope in `errors/forage-problem-types.yml`.
- Rate limit: 900 requests / 5 min per IP → HTTP 429 `too_many_requests`.
