# API Contract

## Use This Reference For

- merchant config setup
- subscription plan setup
- image upload for merchant logo or plan artwork
- third-party checkout session creation
- session query and reconciliation
- recurring subscription query, cancel, and resume
- signed callback verification
- fallback manual completion flows

## Bearer Auth

Use this when the human user asks how to authenticate third-party requests.

- Header:
  - `Authorization: Bearer {portaly_vibe_payment_api_key}`
- Notes:
  - the API key is tied to one `profileId`
  - each key has a fixed `mode`: `live` or `test`
  - live keys start with `pcs_live_`, test keys start with `pcs_test_`
  - mode is derived from the key; it is not passed per-request
  - do not send `profileId` in write requests that derive it from the key

## API Key Creation

Use this when the human user is creating a new API key from the Portaly admin panel.

- Endpoint:
  - `POST /api/creator-subscription/api-keys`
- Required auth:
  - Firebase auth (Portaly admin panel only — not a third-party API)
- Request fields:
  - `profileId`: required
  - `mode`: optional, `live` (default) or `test`
- Response fields:
  - `data.apiKey.id`
  - `data.apiKey.profileId`
  - `data.apiKey.status`
  - `data.apiKey.mode`
  - `data.apiKey.keyPrefix`
  - `data.apiKey.callbackSecret` (masked)
  - `data.secret` (full API key — shown only once at creation)

Mode behavior:

- `live` keys use prefix `pcs_live_` and connect to production payment providers
- `test` keys use prefix `pcs_test_` and connect to sandbox payment providers (e.g., TapPay sandbox)
- Test mode orders are stored in a separate `sandboxOrders` collection
- A single `profileId` can have both a live and a test key active simultaneously
- Mode is fixed at creation time and cannot be changed

## Merchant Config

Use this when the human user needs to set or update merchant branding for Portaly Vibe Payment.

- Read endpoint:
  - `GET /api/creator-subscription/config?profileId={profileId}`
- Setup endpoints:
  - `PUT /api/creator-subscription/config`
  - `POST /api/creator-subscription/config/images`
- Setup headers:
  - `Authorization: Bearer {portaly_vibe_payment_api_key}`

`PUT /api/creator-subscription/config`

- Update endpoint for merchant profile config, currently only supports `merchantName`. Image should upload via `/api/creator-subscription/config/images`.

- Request fields:
  - `merchantName`: optional string value

- Request body:
```json
{
  "merchantName": "Example Merchant",
}
```

- Response fields:
  - `data.profileId`
  - `data.merchantName`
  - `data.merchantLogo`
  - `data.updatedAt`

`POST /api/creator-subscription/config/images`

- Content type:
  - `multipart/form-data`
- Form fields:
  - `file`: required image file
  - `filename`: optional override filename
- Supported image types:
  - `image/jpeg`
  - `image/png`
  - `image/webp`
  - `image/gif`
- Size limit:
  - 8 MB
- Response fields:
  - `data.merchantLogo`
  - `data.file.id`
  - `data.file.publicURL`
  - `data.creatorSubscriptionConfig`

## Subscription Plans

Use this when the human user wants the Agent to create or maintain the product basics that will be listed on Portaly.

- **Plans belong to the `profileId` and are shared across live and test modes.** Always query existing plans before creating a new one to avoid duplicates.
- Read endpoints:
  - `GET /api/creator-subscription/plans?profileId={profileId}`
  - `GET /api/creator-subscription/plans/{planId}`
- Setup endpoints:
  - `POST /api/creator-subscription/plans`
  - `PUT /api/creator-subscription/plans/{planId}`
  - `POST /api/creator-subscription/plans/{planId}/images`
- Setup headers:
  - `Authorization: Bearer {portaly_vibe_payment_api_key}`

`POST /api/creator-subscription/plans`

- Request fields:
  - `name`: required
  - `description`: optional
  - `amount`: required positive number for fixed pricing; omit or set to `0` for dynamic pricing
  - `currency`: optional, defaults to `TWD`
  - `billingPeriod`: required, `monthly` or `yearly`, `one-time` is a single-payment plan that does not auto-renew
  - `pricingType`: optional, `fixed` (default) or `dynamic`. Dynamic pricing plans must use `one-time` billing period; the actual amount is set per checkout session
  - `status`: optional, `active` or `inactive`
  - `merchantPlanId`: optional merchant-side product id
  - `externalInformationUrl`: optional object with `url` and `text`
- Request body (fixed pricing):

```json
{
  "name": "Pro Monthly",
  "description": "Monthly subscription plan",
  "amount": 299,
  "currency": "TWD",
  "billingPeriod": "monthly",
  "status": "active",
  "merchantPlanId": "merchant_plan_monthly_001",
  "externalInformationUrl": {
    "url": "https://example.com/plan-details",
    "text": "Learn more"
  }
}
```

- Request body (dynamic pricing):

```json
{
  "name": "Custom Payment",
  "description": "Pay any amount",
  "pricingType": "dynamic",
  "billingPeriod": "one-time",
  "status": "active",
  "merchantPlanId": "merchant_plan_dynamic_001"
}
```

- Response fields:
  - `data.id`
  - `data.profileId`
  - `data.name`
  - `data.amount`
  - `data.currency`
  - `data.billingPeriod`
  - `data.pricingType`
  - `data.status`
  - `data.image`
  - `data.createdAt`
  - `data.updatedAt`
- **Encoding note:** On Windows, if `data.name` or `data.description` contains garbled text, fix the shell encoding and use `PUT /api/creator-subscription/plans/{planId}` to correct it.

`PUT /api/creator-subscription/plans/{planId}`

- Request fields:
  - `profileId`: required and must match the plan owner
  - `name`: optional
  - `description`: optional
  - `amount`: optional positive number (must be non-negative for dynamic pricing plans)
  - `currency`: optional
  - `billingPeriod`: optional, `monthly` or `yearly`, `one-time` is a single-payment plan that does not auto-renew
  - `pricingType`: optional, `fixed` or `dynamic`
  - `status`: optional, `active` or `inactive`
  - `merchantPlanId`: optional

`POST /api/creator-subscription/plans/{planId}/images`

- Content type:
  - `multipart/form-data`
- Form fields:
  - `file`: required image file
  - `filename`: optional override filename
- Supported image types:
  - `image/jpeg`
  - `image/png`
  - `image/webp`
  - `image/gif`
- Size limit:
  - 8 MB
- Response fields:
  - `data.image`
  - `data.file.id`
  - `data.file.publicURL`
  - `data.plan`

## Session Creation

Use this when the human user needs to send the buyer into Portaly hosted checkout.

- Endpoint:
  - `POST /api/creator-subscription/checkout-sessions`
- Required headers:
  - `Authorization: Bearer {portaly_vibe_payment_api_key}`
  - `Content-Type: application/json`
- Request fields:
  - `planId`: Portaly plan id
  - `amount`: optional positive number. **Required** for dynamic pricing plans; ignored for fixed pricing plans
  - `successRedirectUrl`: optional merchant success page
  - `cancelRedirectUrl`: optional merchant cancel page
  - `callbackUrl`: optional merchant callback endpoint
  - `merchantOrderNumber`: optional merchant-side order id
  - `metadata`: optional string-keyed extra context

Request body (fixed pricing plan):

```json
{
  "planId": "plan_123",
  "successRedirectUrl": "https://merchant.example/success",
  "cancelRedirectUrl": "https://merchant.example/cancel",
  "callbackUrl": "https://merchant.example/api/portaly/callback",
  "merchantOrderNumber": "order_001",
  "metadata": {
    "source": "web",
    "cartId": "cart_123"
  }
}
```

Request body (dynamic pricing plan):

```json
{
  "planId": "plan_dynamic_456",
  "amount": 500,
  "successRedirectUrl": "https://merchant.example/success",
  "cancelRedirectUrl": "https://merchant.example/cancel",
  "callbackUrl": "https://merchant.example/api/portaly/callback",
  "merchantOrderNumber": "order_002",
  "metadata": {
    "source": "web"
  }
}
```

- Response fields:
  - `data.sessionId`: Portaly checkout session id
  - `data.status`: initial status, usually `checkout_ready`
  - `data.checkoutUrl`: URL the buyer should visit
  - `data.checkoutToken`: server-side token for provider routes or manual completion
  - `data.expiresAt`: session expiry timestamp

```json
{
  "data": {
    "sessionId": "session_123",
    "status": "checkout_ready",
    "checkoutUrl": "https://payment-host/checkout/subscription/session_123",
    "checkoutToken": "hex_token",
    "expiresAt": "2026-03-20T12:30:00.000Z"
  }
}
```

- Integration notes:
  - persist `checkoutUrl`, `sessionId`, `checkoutToken`, and `expiresAt`
  - redirect the buyer to `checkoutUrl`
  - `callbackSecret` is not passed in the request; Portaly derives it from the authorized API key
  - the session inherits `mode` from the API key used to create it (`live` or `test`)
  - current implementation contract: `subscriptionId === checkoutSessionId === sessionId`
  - for recurring subscriptions, persist `sessionId` as the subscription identifier used by cancel or resume APIs
  - for dynamic pricing plans, include `amount` in the request body; omitting it returns a 400 error

Node.js example:

```js
const response = await fetch(
  "https://portaly.example/api/creator-subscription/checkout-sessions",
  {
    method: "POST",
    headers: {
      "content-type": "application/json",
      authorization: `Bearer ${process.env.PORTALY_VIBE_PAYMENT_API_KEY}`,
    },
    body: JSON.stringify({
      planId: "plan_123",
      successRedirectUrl: "https://merchant.example/success",
      cancelRedirectUrl: "https://merchant.example/cancel",
      callbackUrl: "https://merchant.example/api/portaly/callback",
      merchantOrderNumber: "order_001",
      metadata: {
        source: "web",
        cartId: "cart_123",
      },
    }),
  }
);

const result = await response.json();
const { sessionId, checkoutUrl, checkoutToken, expiresAt } = result.data;
```

## Session Query

Use this when the human user needs reconciliation or a status page.

- Endpoint:
  - `GET /api/creator-subscription/checkout-sessions/{sessionId}`
- Required headers:
  - `Authorization: Bearer {portaly_vibe_payment_api_key}`
- Useful response fields:
  - `status`
  - `merchantOrderNumber`
  - `customerEmail`
  - `metadata`
  - `expiresAt`
  - `completedAt`
- Common uses:
  - merchant status pages
  - reconciliation jobs
  - callback retry fallback

## Subscription Query And Lifecycle

Use this when the human user needs to look up a recurring subscription or stop or resume future renewals.

- Endpoints:
  - `GET /api/creator-subscription/subscriptions/{subscriptionId}`
  - `POST /api/creator-subscription/subscriptions/{subscriptionId}/cancel`
  - `POST /api/creator-subscription/subscriptions/{subscriptionId}/resume`
- Required headers:
  - `Authorization: Bearer {portaly_vibe_payment_api_key}`

Current identifier contract:

- `subscriptionId === checkoutSessionId === sessionId`
- If the merchant only has the checkout completed callback payload, it may use `sessionId` directly as `subscriptionId`

`GET /api/creator-subscription/subscriptions/{subscriptionId}`

- Useful response fields:
  - `id`
  - `profileId`
  - `planId`
  - `mode` (`live` or `test`)
  - `billingPeriod`
  - `status`
  - `cancelAtPeriodEnd`
  - `nextBillingAt`
  - `cancelRequestedAt`
  - `cancelEffectiveAt`
  - `canceledAt`

`POST /api/creator-subscription/subscriptions/{subscriptionId}/cancel`

- Supported only for recurring subscriptions:
  - `billingPeriod = monthly`
  - `billingPeriod = yearly`
- Behavior:
  - stops the next recurring charge
  - does not issue a refund
  - keeps the current paid period active until `cancelEffectiveAt`
- Request fields:
  - `reason`: optional, one of `customer_requested | payment_failures | manual | other`
  - `reasonNote`: optional string

```json
{
  "reason": "customer_requested",
  "reasonNote": "Customer asked to stop renewal"
}
```

- Useful response fields:
  - `id`
  - `status`
  - `cancelAtPeriodEnd`
  - `cancelRequestedAt`
  - `cancelEffectiveAt`
  - `nextBillingAt`

`POST /api/creator-subscription/subscriptions/{subscriptionId}/resume`

- Supported only when the recurring subscription is pending end-of-period cancellation
- Behavior:
  - clears the pending cancellation
  - allows future recurring renewal to continue

```json
{}
```

- Useful response fields:
  - `id`
  - `status`
  - `cancelAtPeriodEnd`
  - `nextBillingAt`

Common validation notes:

- `one-time` plans do not support cancel or resume
- a fully `canceled` subscription cannot be resumed
- these lifecycle APIs are merchant-system APIs and do not use Firebase auth

## Manual Completion

Use this only for controlled recovery or non-hosted payment flows.

- Endpoint:
  - `POST /api/creator-subscription/checkout-sessions/{sessionId}/complete`
- Required headers:
  - `Authorization: Bearer {portaly_vibe_payment_api_key}`
  - `Content-Type: application/json`
- Request fields:
  - `checkoutToken`
  - `customerName`
  - `customerEmail`
  - `paymentReference`
  - `paymentMethod`
  - `paidAmount`
  - `status`
  - `failureReason`: optional when status is `failed`

Request body:

```json
{
  "checkoutToken": "hex_token",
  "customerName": "David",
  "customerEmail": "david@example.com",
  "paymentReference": "txn_123",
  "paymentMethod": "custom-provider",
  "paidAmount": 299,
  "status": "completed"
}
```

Allowed `status` values:

- `completed`
- `failed`
- `canceled`

## Signed Callback

Use this when the human user needs to verify Portaly callback requests.

- Headers:
  - `x-portaly-event`
  - `x-portaly-timestamp`
  - `x-portaly-signature`
- Payload fields to persist:
  - `sessionId`
  - `subscriptionId` if present
  - `mode` (`live` or `test`)
  - `merchantOrderNumber`
  - `status`
  - `paymentReference`
  - `paymentMethod`
  - `customerEmail`
  - `completedAt`

Payload example:

```json
{
  "event": "creator_subscription.checkout.completed",
  "sessionId": "session_123",
  "subscriptionId": "session_123",
  "profileId": "profile_123",
  "planId": "plan_123",
  "mode": "live",
  "status": "completed",
  "merchantOrderNumber": "order_001",
  "amount": 299,
  "currency": "TWD",
  "customerEmail": "buyer@example.com",
  "customerName": "Buyer",
  "invoice": {
    "type": "b2c",
    "carrierType": "phone",
    "carrierNumber": "/ABCDE12"
  },
  "completedAt": "2026-03-12T10:05:00.000Z",
  "metadata": {
    "source": "web"
  },
  "paymentReference": "txn_123456",
  "paymentMethod": "tappay"
}
```

Verification rule:

- base string: `{timestamp}.{stable_json(payload)}`
- algorithm: `HMAC-SHA256`
- secret: the API key's `callbackSecret`

Callback notes:

- current implementation contract: `subscriptionId === sessionId`
- if the callback payload consumed by the merchant side does not explicitly expose `subscriptionId`, the merchant may safely persist `sessionId` as the recurring subscription identifier
- use `sessionId` as the idempotency key for callback processing
- the `mode` field indicates whether this callback originated from a live or test checkout; merchants should use it to route test callbacks to sandbox order handling

Use `scripts/sign_callback.mjs` for Node.js/TypeScript-oriented work and `scripts/sign_callback.py` for a language-agnostic reference.

Express-style callback example:

```js
import express from "express";
import { verifyPortalyCallback } from "../portaly/sign_callback.mjs";

const app = express();
app.use(express.json());

app.post("/api/portaly/callback", (req, res) => {
  const timestamp = req.header("x-portaly-timestamp") || "";
  const signature = req.header("x-portaly-signature") || "";

  const verified = verifyPortalyCallback({
    secret: process.env.PORTALY_CALLBACK_SECRET,
    timestamp,
    payload: req.body,
    signature,
  });

  if (!verified) {
    return res.status(401).json({ error: "invalid signature" });
  }

  const {
    sessionId,
    subscriptionId = sessionId,
    merchantOrderNumber,
    status,
    paymentReference,
  } = req.body;

  // Persist the verified callback and reconcile your local order state here.
  console.log({
    sessionId,
    subscriptionId,
    merchantOrderNumber,
    status,
    paymentReference,
  });

  return res.status(200).json({ ok: true });
});
```

## External Ownership Split

Third party usually owns:

- plan selection UI
- session creation API call
- storing merchant order context
- success/cancel pages
- callback receiver and reconciliation

Portaly owns:

- hosted checkout page
- email verification
- TapPay and 91APP payment initiation/finalization
- subscription and payment persistence
- invoice task creation
- bridge order creation
