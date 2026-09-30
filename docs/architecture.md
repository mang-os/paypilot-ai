# Architecture

PayPilot AI separates advisory catalog discovery from checkout pricing and policy decisions. The Next.js interface calls FastAPI; SQLAlchemy models store catalog, agent, mandate, checkout, order, transaction and audit state.

## Components

| Component | Responsibility | Source |
| --- | --- | --- |
| Checkout routers | Create/update checkouts and request order creation | [acp_checkouts.py](../razorpay-agentic-commerce/backend/app/routers/acp_checkouts.py) |
| Pricing | Derive totals from catalog and offer records | [checkout_engine.py](../razorpay-agentic-commerce/backend/app/services/checkout_engine.py) |
| Policy | Check agent status, stock, limits and mandate validity | [policy_engine.py](../razorpay-agentic-commerce/backend/app/services/policy_engine.py) |
| Mandate signatures | Sign and verify spending authorization payloads | [mandate_service.py](../razorpay-agentic-commerce/backend/app/services/mandate_service.py) |
| Payment integration | Create provider or mock orders and check webhook HMAC | [razorpay_service.py](../razorpay-agentic-commerce/backend/app/services/razorpay_service.py) |
| Payment events | Update orders, transactions, checkout and inventory | [webhooks.py](../razorpay-agentic-commerce/backend/app/routers/webhooks.py) |

## Data ownership

The backend recomputes subtotal, discount, tax and final amount from product/offer records. Product discovery does not authorize a charge. Policy checks run before the code reserves inventory and asks Razorpay to create an order.

Order creation returns an order reference. Completion occurs in the signed webhook handler, not simply when an order exists. Audit events expose the sequence to the interface.

## Deployment modes

[Compose](../razorpay-agentic-commerce/docker-compose.yml) starts PostgreSQL, backend migrations/demo seeding and the Next.js frontend. [Configuration](../razorpay-agentic-commerce/backend/app/config.py) defaults to SQLite for local non-Compose development.

Without Razorpay credentials, order creation generates mock IDs. Without the configured Groq key/model, catalog discovery uses the deterministic fallback described in the application guide.

## Design limits

This is not a distributed transaction across database state and Razorpay. Checkout completion has multiple commits, and current inventory/mandate operations do not prove concurrency-safe allocation. See [payment boundaries](payment-boundaries.md) and [verification](verification.md) before interpreting the demo as a stronger guarantee.
