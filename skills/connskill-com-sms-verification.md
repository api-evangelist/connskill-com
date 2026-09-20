---
generated: '2026-09-19'
method: generated
name: Rent an SMS verification number and read the code
description: Quote, buy and poll one SMS verification number with x402, and cancel it while it is still unused.
api: openapi/connskill-com-openapi.yml
operations: [smsServices, smsPricing, smsQuote, smsOrder, smsStatus, smsCancel]
source: >-
  Grounded in openapi/connskill-com-openapi.yml (operationIds verified verbatim), the A2A card skills
  sms-order / sms-status / sms-cancel, conventions/connskill-com-conventions.yml and errors/connskill-com-problem-types.yml.
---

# Rent an SMS verification number and read the code

One-off phone verification for a service (Telegram, WhatsApp, Discord, ...) paid per call in USDC on Base. No account, no API key.

## Auth and payment
- Free routes need nothing. The paid route answers `402` with the x402 challenge; pay the exact `accepts[0].amount` and repeat the **same** request with `PAYMENT-SIGNATURE`. See `conventions/connskill-com-conventions.yml`.
- The order id returned by `smsOrder` is the secret for every follow-up call: keep it private.

## Steps
1. **List services and countries** — `smsServices` (`GET /sms/v1/sms-services`) for valid `service` and `country` codes.
2. **Find the cheapest country** — `smsPricing` (`GET /sms/v1/sms-pricing`) for one service, then **quote** — `smsQuote` (`GET /sms/v1/sms-quote`) with `service` + `country`. Show the price, network, asset and recipient to the user and get approval before paying.
3. **Buy the number** — `smsOrder` (`POST /sms/v1/sms-order`, body `{"service": "...", "country": "US"}`). Variable price from 0.25 USDC up to 3 USDC; the 402 challenge binds the exact amount.
4. **Poll for the code** — `smsStatus` (`POST /sms/v1/sms-status`, body `{"orderId": "..."}`), free.
5. **If no code arrives** — `smsCancel` (`POST /sms/v1/sms-cancel`, body `{"orderId": "..."}`), free. Verbatim window: "Cancels a rented number that received no code yet (provider-side refund of our cost; the x402 payment itself is not refunded)."

## Errors and retries
- A timeout after payment is an **unknown outcome**: do not pay again. Request private redelivery with a wallet proof (`paymentWalletChallenge`, purpose `redelivery`) or open wallet support (`supportChallenge` -> `supportTicketCreate` with the `purchaseId`). `504 stateUnclear` means exactly this.
- Envelope is `{"error": "<code>", "hint": "<text>"}`; see `errors/connskill-com-problem-types.yml`.

## Notes
- Use only for accounts you are authorised to verify; the provider forwards phone orders to SMSPool or 5sim (privacy page).
