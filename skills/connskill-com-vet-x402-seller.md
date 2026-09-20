---
generated: '2026-09-19'
method: generated
name: Vet an x402 seller before paying it
description: Read the published conformance battery for a seller, then buy one trust check and one domain security check before authorising any payment to it.
api: openapi/connskill-com-openapi.yml
operations: [conformanceReports, trustCheck, domainSecurityCheck, chainState]
source: >-
  Grounded in openapi/connskill-com-openapi.yml (operationIds verified verbatim), https://agent.connskill.com/trust,
  https://agent.connskill.com/conformance and conventions/connskill-com-conventions.yml.
---

# Vet an x402 seller before paying it

An agent that pays strangers per call needs to tell a working seller from a listing. CONNSKILL publishes an HTTP-refusal battery for x402 origins and sells two paid checks.

## Auth and payment
- `conformanceReports` is free. `trustCheck` (0.05 USDC), `domainSecurityCheck` (0.05 USDC) and `chainState` (0.05 USDC) are paid via x402; each is eligible for one free sample per UTC day with the header `x-free-sample: 1`.

## Steps
1. **Read the published battery** — `conformanceReports` (`GET /v1/conformance?origin=<seller-host>`). 404 means no report yet. A clean 11/11 shows the seller refuses altered, unsigned payments; it does not prove delivery or cryptographic isolation (provider's own disclaimer).
2. **Probe the live seller** — `trustCheck` (`POST /v1/trust-check`, body `{"origin": "<seller-host>"}`): probes the live 402, reads USDC inflow to its `payTo` on Base, counts payers and flags self-dealing. Result is green / yellow / red with reasons and the window it could read.
3. **Check the domain** — `domainSecurityCheck` (`POST /v1/domain-security-check`, body `{"domain": "<seller-domain>"}`): TLS, HSTS/CSP, SPF/DMARC, CAA, DNSSEC, RDAP expiry, leak exposure, graded A-F.
4. **Optional: inspect the recipient wallet** — `chainState` (`POST /chain/v1/chain-state`) for balances and EOA/contract status of the seller's `payTo` (read-only; no custody).
5. Decide with the user: set a per-call cap (the npm client's `X402_MAX_USD`) before enabling payment to that seller.

## Errors and retries
- Paid checks answer `402` first; repeat the same request with `PAYMENT-SIGNATURE`. Never re-pay after a timeout — redeliver or open wallet support. See `errors/connskill-com-problem-types.yml`.

## Notes
- Reports "cover only the stated tests and time" (https://agent.connskill.com/disclaimer). On-chain simulations are not investment advice.
