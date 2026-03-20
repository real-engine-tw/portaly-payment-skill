# Checkout And Renewal

This reference is supplemental background. It is not part of the core third-party API contract.

## Use This Reference For

- explaining provider-specific behavior after checkout starts
- mapping checkout status transitions
- understanding what Portaly writes after first payment
- answering recurring charge, invoice, payout, or order-bridge questions

## Hosted Checkout Flow

1. Merchant backend creates a checkout session with a Portaly Vibe Payment API key.
2. Portaly returns `checkoutUrl`, `sessionId`, `checkoutToken`, and expiry.
3. Buyer opens `/checkout/subscription/{sessionId}` on Portaly.
4. Buyer submits email and completes email verification.
5. Buyer pays with TapPay or 91APP.
6. Portaly completes the session and dispatches the signed callback.

## Email Verification Rules

- Email verification is mandatory before payment.
- The code is 6 digits.
- The code expires after 5 minutes.
- One email can receive at most 3 messages per day in `Asia/Taipei`.
- Brevo template `#607` is used.

## Provider Split

### TapPay

- Frontend creates `prime`.
- Portaly server charges with `pay-by-prime`.
- First charge is synchronous.
- On success, Portaly stores a payment method secret with:
  - `card_key`
  - `card_token`
- Then Portaly completes the checkout session and writes subscription side effects immediately.

### 91APP

- Frontend creates `txnToken`.
- Portaly server creates a payment request and sets the session to `initiated`.
- Buyer is redirected into 91APP.
- 91APP later calls Portaly callback.
- Portaly queries the transaction, stores `cardToken`, then finalizes the session.
- `merchantConsumerId` is used as the recurring identifier.

## Session Status Meanings

- `checkout_ready`: session exists and can be paid
- `initiated`: provider flow started and Portaly is waiting for completion
- `completed`: payment finalized and downstream records were written
- `failed`: provider reported failure or finalize step failed
- `expired`: session TTL elapsed

## Portaly Side Effects On First Successful Payment

Portaly writes:

- `creatorSubscriptionPaymentMethods/{paymentMethodId}`
- `creatorSubscriptions/{checkoutSessionId}`
- `creatorSubscriptionPayments/{paymentId}`
- `creatorSubscriptionInvoiceTasks/{paymentId}`
- `orders/creatorSubscription_{paymentId}`

Important conventions:

- `paymentMethodId = checkoutSessionId`
- `creatorSubscriptionId = checkoutSessionId`
- the order bridge links Portaly Vibe Payment revenue into the existing `orders` pipeline

## Order Bridge Notes

The bridge order calculates:

- platform fee: `4%`
- tax fee: `1%` for company, `6%` for personal
- `netTotal = amount - feeAmount - taxFeeAmount`

The order document also stores `merchantOrderNumber`, `walletAccount`, `paymentMethod`, and metadata.

## Renewal Behavior

Recurring charges are split by stored payment method:

- TapPay uses token-based charging
- 91APP uses `request-by-cardToken`

On successful renewal, Portaly writes:

- a new `creatorSubscriptionPayments` record
- a new `creatorSubscriptionInvoiceTasks` record
- a new bridge `orders` record

When answering renewal questions, be explicit that recurring behavior depends on the payment method secret saved during the first successful checkout.
