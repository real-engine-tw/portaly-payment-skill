# API Contract

## Use This Reference For

- third-party checkout session creation
- session query and reconciliation
- signed callback verification
- fallback manual completion flows

## Bearer Auth

External systems authenticate with:

```http
Authorization: Bearer {creator_subscription_api_key}
```

The API key is tied to one `profileId`. Do not send `profileId` in write requests that derive it from the key.

## Session Creation

Endpoint:

```http
POST /api/creator-subscription/checkout-sessions
```

Request body:

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

Response shape:

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

Implementation notes:

- `checkoutUrl` is the link the buyer should visit.
- `checkoutToken` is needed by Portaly-hosted payment routes and manual completion. Persist it on the server side.
- `callbackSecret` is not passed in the request; Portaly derives it from the authorized API key.

Node.js example:

```js
const response = await fetch(
  "https://portaly.example/api/creator-subscription/checkout-sessions",
  {
    method: "POST",
    headers: {
      "content-type": "application/json",
      authorization: `Bearer ${process.env.PORTALY_CREATOR_SUBSCRIPTION_API_KEY}`,
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

Endpoint:

```http
GET /api/creator-subscription/checkout-sessions/{sessionId}
```

Useful returned fields:

- `status`
- `merchantOrderNumber`
- `customerEmail`
- `metadata`
- `expiresAt`
- `completedAt`

Use this for merchant status pages, reconciliation jobs, or callback retry fallback.

## Manual Completion

Endpoint:

```http
POST /api/creator-subscription/checkout-sessions/{sessionId}/complete
```

Use this only for controlled recovery or non-hosted payment flows.

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

Headers:

- `x-portaly-event`
- `x-portaly-timestamp`
- `x-portaly-signature`

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
