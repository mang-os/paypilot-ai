# Payment boundaries

## What the implementation checks

- Checkout totals come from live product/offer records at creation, update and completion.
- Policy checks evaluate agent status, inventory, transaction/daily limits, checkout state, mandate ownership/status/expiry/amount and the stored signed payload.
- Razorpay order creation occurs after policy approval and stock reservation.
- Order creation retries selected transient failures. Exhausted timeout handling releases inventory, reactivates the mandate and leaves the checkout retryable.
- Webhook processing checks HMAC-SHA256 against the raw request body before processing payment events.
- Duplicate completion returns the existing order when its reference is already stored. Webhook handling looks for an existing transaction by payment ID.

## What those checks do not establish

A valid webhook signature is not an explicit comparison of captured amount/currency with checkout totals: the current handler does not perform that comparison. An existing order is not payment completion. Mock order creation does not move money.

Checking stock and subsequently changing it across separate commits is not an atomic allocation contract. Daily spending checks also must not be advertised as proof against concurrent in-flight purchases. An application-level duplicate check alone does not prove every concurrent delivery or crash window is handled.

Pricing converts database amounts to Python floats and uses a configurable flat tax rate. Demo key handling, deployment authentication, failure reconciliation and external payment uncertainty require further review for production use.

## Source

[Checkout router](../razorpay-agentic-commerce/backend/app/routers/acp_checkouts.py) · [Policy engine](../razorpay-agentic-commerce/backend/app/services/policy_engine.py) · [Pricing](../razorpay-agentic-commerce/backend/app/services/checkout_engine.py) · [Payment integration](../razorpay-agentic-commerce/backend/app/services/razorpay_service.py) · [Webhooks](../razorpay-agentic-commerce/backend/app/routers/webhooks.py)
