---
name: monetize-web-app
description: Design or integrate ethical monetization for a React web application, including subscriptions, one-time payments, usage limits, entitlements, and billing status. Use when the user asks to monetize a web app.
argument-hint: "[business model or billing task]"
license: MIT
metadata:
  tags: [react, billing, subscriptions, entitlements, analytics]
---

# Monetize Web App

Keep billing decisions server-authoritative and transparent.

- Define plans, entitlements, trial, renewal, cancellation, refund, tax, and failed-payment states.
- Use the payment provider's hosted or tokenized flow; never collect raw card data in the app.
- Verify webhooks idempotently and derive access from server state.
- Make pricing, limits, renewal date, and cancellation path clear and accessible.
- Add analytics for funnel health without leaking sensitive payment data.
- Test duplicate webhooks, expired access, offline return, and failed checkout.

## Full Workflow

1. Define the business outcome, plans, entitlements, limits, trial, renewal, cancellation, refund, tax, and support path.
2. Inspect existing auth, user/account model, server boundaries, analytics, and payment provider conventions.
3. Write contract tests for checkout creation, webhook idempotency, access changes, expiration, and failed payment.
4. Implement provider adapters and server-authoritative entitlement state before connecting React pricing and billing UI.
5. Render transparent pricing, limits, renewal date, cancellation, pending, success, failure, and offline return states.
6. Verify webhook replay, direct API access, stale client state, privacy, accessibility, and sensitive-data handling.
7. Document provider assumptions, operational alerts, reconciliation, rollback, and user-facing limitations.