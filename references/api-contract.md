# API Contract

## Use This Reference For

- merchant config setup
- subscription plan setup
- image upload for merchant logo or plan artwork
- third-party checkout session creation
- session query and reconciliation
- signed callback verification
- fallback manual completion flows

## Bearer Auth

Use this when the human user asks how to authenticate third-party requests.

- Header:
  - `Authorization: Bearer {portaly_vibe_payment_api_key}`
- Notes:
  - the API key is tied to one `profileId`
  - do not send `profileId` in write requests that derive it from the key

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

- Request fields:
  - `merchantLogo`: optional `images/{fileId}` value
- Request body:

```json
{
  "merchantLogo": "images/file_123"
}
```

- Response fields:
  - `data.profileId`
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
  - `amount`: required positive number
  - `currency`: optional, defaults to `TWD`
  - `billingPeriod`: required, `monthly` or `yearly`, `one-time` is a single-payment plan that does not auto-renew
  - `status`: optional, `active` or `inactive`
  - `merchantPlanId`: optional merchant-side product id
  - `externalInformationUrl`: optional object with `url` and `text`
- Request body:

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

- Response fields:
  - `data.id`
  - `data.profileId`
  - `data.name`
  - `data.amount`
  - `data.currency`
  - `data.billingPeriod`
  - `data.status`
  - `data.image`
  - `data.createdAt`
  - `data.updatedAt`

`PUT /api/creator-subscription/plans/{planId}`

- Request fields:
  - `profileId`: required and must match the plan owner
  - `name`: optional
  - `description`: optional
  - `amount`: optional positive number
  - `currency`: optional
  - `billingPeriod`: optional, `monthly` or `yearly`, `one-time` is a single-payment plan that does not auto-renew
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
  - `successRedirectUrl`: optional merchant success page
  - `cancelRedirectUrl`: optional merchant cancel page
  - `callbackUrl`: optional merchant callback endpoint
  - `merchantOrderNumber`: optional merchant-side order id
  - `metadata`: optional string-keyed extra context

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
  "profileId": "profile_123",
  "planId": "plan_123",
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

  const { sessionId, merchantOrderNumber, status, paymentReference } = req.body;

  // Persist the verified callback and reconcile your local order state here.
  console.log({ sessionId, merchantOrderNumber, status, paymentReference });

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
