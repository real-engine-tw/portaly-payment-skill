---
name: portaly-creator-subscription-integration
description: Implement or review Portaly creator subscription integrations for third-party systems. Use when Codex needs to connect an external product to Portaly hosted checkout, create checkout sessions with creator subscription API keys, wire TapPay or 91APP payment initiation, verify Portaly signed callbacks, handle redirect and callback flows, or explain the end-to-end creator subscription payment and renewal pipeline.
---

# Portaly Creator Subscription Integration

Build around Portaly's hosted checkout instead of rebuilding payment orchestration. Treat Portaly as the system that owns checkout session state, email verification, first charge finalization, recurring payment method storage, subscription creation, payment records, invoice task creation, and callback dispatch.

## Quick Start

1. Confirm the integration target.
   If the user needs to launch a creator subscription checkout from an external service, start with `references/api-contract.md`.
   If the user needs to understand provider behavior, order bridge records, or renewal side effects, read `references/checkout-and-renewal.md`.
2. Identify which part of the flow is owned by the third party.
   Usually the third party owns plan selection, checkout session creation, success/cancel pages, and callback consumption.
   Portaly owns the hosted checkout UI and payment persistence.
3. Produce concrete artifacts.
   Prefer returning request/response examples, callback verification code, environment checklists, and a short sequence diagram or implementation checklist.

## Workflow

### 1. Apply for the API key

- Require a creator subscription API key bound to the target `profileId`.
- Instruct the human user to apply for or create the creator subscription API key in the Portaly admin at `https://portaly.cc`.
- Be explicit that this step is performed by a human operator in Portaly admin panel, not by the third-party integration code.
- Tell the human user to store the issued secret material safely, or store it on the user's behalf only in an appropriate secret manager or secure environment store.
- Tell the integrator to preserve both:
  - the full API key for bearer auth
  - the API key's `callbackSecret` for signature verification
- Do not leave the API key or `callbackSecret` in chat transcripts, source files, client-side code, screenshots, or plaintext docs.

### 2. Create a valid subscription plan

- Require at least one active plan in Portaly before creating a checkout session.
- Confirm the plan is active and that its amount, currency, and billing period match the third-party product being sold.
- If the third party has its own product catalog, persist the Portaly `planId` together with the merchant's internal product or entitlement identifier.
- Treat the `checkoutUrl` returned by Portaly as authoritative. Do not reconstruct it from guessed domains.

### 3. Create the checkout session

- Call `POST /api/creator-subscription/checkout-sessions` with `Authorization: Bearer {api_key}`.
- Send `planId` and optional `successRedirectUrl`, `cancelRedirectUrl`, `callbackUrl`, `merchantOrderNumber`, and string-keyed `metadata`.
- Persist `sessionId`, `checkoutToken`, `checkoutUrl`, and `expiresAt` on the third-party side.
- Redirect the buyer to `checkoutUrl`.

### 4. Let Portaly run hosted checkout

- Portaly checkout enforces email verification before payment.
- TapPay completes synchronously in the checkout request.
- 91APP uses `initiate -> redirect -> provider callback -> finalize`.
- Do not ask the third party to collect card tokens directly unless the task is specifically about Portaly's internal checkout implementation.

### 5. Consume the result

- The primary external confirmation is the signed callback to `callbackUrl`.
- Optionally poll `GET /api/creator-subscription/checkout-sessions/{sessionId}` for status pages or reconciliation.
- Use manual `POST /api/creator-subscription/checkout-sessions/{sessionId}/complete` only as an exception flow when the user is building a non-hosted or recovery flow.

### 6. Verify and persist

- Verify `x-portaly-signature` with the API key's `callbackSecret`.
- Use the exact timestamp from `x-portaly-timestamp`.
- Serialize the callback payload with stable key ordering before HMAC.
- Reference implementations live in `scripts/sign_callback.py` and `scripts/sign_callback.mjs`.
- After verification, persist `sessionId`, `merchantOrderNumber`, `paymentReference`, `paymentMethod`, `status`, and the raw callback body for auditing.

## Guardrails

- Prefer the hosted checkout flow whenever possible. It already handles email verification, payment-method persistence, callback dispatch, subscription creation, payment creation, invoice task creation, and order bridge writes.
- Do not invent provider behavior. TapPay and 91APP differ materially.
- Do not assume callback delivery means success without checking the `status` and verified signature.
- Do not derive subscription state from redirect success pages alone. Redirects are UX only; callback or status query is the source of truth.
- If the task includes recurring billing, payout, or invoice follow-up, load `references/checkout-and-renewal.md` before answering.

## Deliverables

When using this skill, aim to return one or more of:

- a minimal integration plan for the external system
- sample `curl` requests for session creation or session query
- callback verification code in the user's stack
- a redirect/callback sequence diagram
- a troubleshooting list keyed by session status and provider

## Resources

- `references/api-contract.md`
  Use for bearer auth, endpoint contract, callback headers, payload fields, and third-party implementation shape.
- `references/checkout-and-renewal.md`
  Use for provider-specific flow, payment-method storage, subscription/payment/order side effects, and renewal behavior.
- `scripts/sign_callback.py`
  Use when you need a deterministic example of Portaly callback signing and verification.
- `scripts/sign_callback.mjs`
  Prefer this for Node.js, JavaScript, TypeScript, Express, or Next.js integrations.
