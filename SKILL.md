---
name: portaly-vibe-payment-integration
description: Help an AI agent quickly assist human users with Portaly Vibe Payment API integrations. Use when Codex needs to guide a human through applying for API keys, preparing subscription plans, creating checkout sessions, handling redirects and callbacks, verifying Portaly signatures, and explaining the request and response shape of the Portaly Vibe Payment APIs in a concise, list-oriented format.
---

# Portaly Vibe Payment Integration

Use this skill to help a human user finish a Portaly Vibe API integration quickly. Keep answers operational: prefer step lists, API request and response bullets, and copy-ready examples over long architecture explanations.

## Portaly Vibe Payment Environments

### API Host

Use the following API host for Portaly Vibe Payment API calls:

- Production: `https://portaly.cc`
- Sandbox: `https://portaly-git-feat-3819-portaly-vibe-real-engine.vercel.app`

### Payment site

Payment site URLs to which buyers are redirected for checkout:

- Production: `https://payment.portaly.cc`
- Sandbox: `https://portaly-vibe.vercel.app`

## Quick Start

1. Confirm what the human user is trying to build.
   Common cases: create checkout sessions, receive payment callbacks, query session status, or explain the full hosted checkout flow.
2. Start with `references/api-contract.md`.
   Use it for endpoint lists, auth, request bodies, response bodies, and callback headers.
3. Load `references/checkout-and-renewal.md` only when needed.
   Use it only as supplemental reference when the human user asks about post-checkout charging, renewal, payout, invoice, or bridge-order behavior.
4. Return implementation-ready output.
   Prefer numbered steps, API endpoint lists, request and response bullets, and Node.js or TypeScript examples.

## Output Style

- Write for an AI agent that is helping a human user complete integration work.
- Lead with the next concrete steps the human should take.
- Use lists for:
  - setup steps
  - API endpoints
  - required headers
  - request fields
  - response fields
  - callback verification steps
- Prefer concise code samples in JavaScript or TypeScript when the user does not ask for another stack.
- Keep Portaly-owned behavior and third-party-owned behavior clearly separated.

## Workflow

### 1. Apply for the API key

- Require a Portaly Vibe Payment API key for this integration.
- Instruct the human user to apply for or create the Portaly Vibe Payment API key in the Portaly admin at `https://portaly.cc`.
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

## Preferred Response Shape

When answering with this skill, prefer this order:

1. Goal summary
2. Human setup steps
3. API list
4. Request fields
5. Response fields
6. Callback handling steps
7. Example code
8. Troubleshooting notes

## Guardrails

- Prefer the hosted checkout flow whenever possible. It already handles email verification, payment-method persistence, callback dispatch, subscription creation, payment creation, invoice task creation, and order bridge writes.
- Do not assume callback delivery means success without checking the `status` and verified signature.
- Do not derive subscription state from redirect success pages alone. Redirects are UX only; callback or status query is the source of truth.
- Treat `references/checkout-and-renewal.md` as non-API background material. Load it only if the task explicitly touches recurring billing, payout, invoice follow-up, or bridge-order behavior.

## Deliverables

When using this skill, aim to return one or more of:

- a minimal step-by-step integration plan for the human user
- a flat list of relevant APIs
- request and response field breakdowns
- callback verification code in the user's stack
- sample `curl`, `fetch`, or TypeScript snippets
- a troubleshooting list keyed by session status and provider

## Resources

- `references/api-contract.md`
  Use for bearer auth, endpoint contract, callback headers, payload fields, and third-party implementation shape.
- `references/checkout-and-renewal.md`
  Use only as optional background for post-checkout charging, payment-method storage, subscription/payment/order side effects, and renewal behavior.
- `scripts/sign_callback.py`
  Use when you need a deterministic example of Portaly callback signing and verification.
- `scripts/sign_callback.mjs`
  Prefer this for Node.js, JavaScript, TypeScript, Express, or Next.js integrations.
