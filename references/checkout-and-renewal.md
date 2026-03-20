# Checkout And Renewal

This reference is supplemental background for third-party integrators. It explains the hosted checkout lifecycle at a high level.

## Use This Reference For

- understanding the overall checkout flow
- explaining what happens after the buyer pays
- clarifying how renewal works

## Checkout Flow

1. The merchant backend creates a checkout session with the Portaly Vibe Payment API.
2. Portaly returns a `checkoutUrl` and `sessionId`.
3. The merchant redirects the buyer to `checkoutUrl`.
4. The buyer completes Portaly hosted checkout.
5. After payment is finalized, Portaly sends the signed callback if `callbackUrl` was provided.
6. The merchant verifies the callback signature and updates its own order or subscription state.

## Important Integration Rules

- Treat `checkoutUrl` as the only buyer entry point for hosted checkout.
- Do not treat the browser redirect alone as the source of truth.
- Use the signed callback or session query result as the final payment status.
- Use `GET /api/creator-subscription/checkout-sessions/{sessionId}` for reconciliation if needed.

## What Portaly Handles

In the hosted flow, Portaly handles:

- buyer checkout experience
- required payment steps such as email verification
- payment completion
- subscription activation after the first successful payment
- recurring charge setup for future renewals

## Renewal Behavior

- If the first checkout succeeds, Portaly keeps the payment method needed for future recurring charges.
- Later renewals are charged by Portaly without requiring the buyer to repeat the full checkout flow.
- On renewal, Portaly updates its own subscription and payment records internally.
- If the merchant needs to reflect renewal results, use Portaly callbacks and/or session or payment reconciliation flows defined by the integration.

## Recommended Third-Party Responsibility

- create the checkout session
- redirect the buyer to Portaly hosted checkout
- verify the signed callback
- persist `sessionId`, merchant order reference, payment status, and callback payload
- use reconciliation queries when callback delivery or buyer state is uncertain

## Scope Note

This document intentionally omits provider-specific payment steps and Portaly internal write details. For the external integration contract, use `references/api-contract.md`.
