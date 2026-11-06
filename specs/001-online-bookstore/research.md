# Architecture Research

## Decisions

### Modular monolith with MVC at the presentation boundary

- **Decision**: Organize the first release as one application with cohesive
  catalog, cart, checkout/order, inventory, account, and fulfillment modules.
  Use MVC for the browser-facing presentation tier and keep application
  use-cases and domain rules behind it. Treat the three tiers as logical
  boundaries, not a deployment mandate.
- **Rationale**: The baseline is one bookstore and the repository has no
  implementation scale or operational constraints that justify distributed
  services. MVC describes presentation responsibilities; it does not replace
  application and domain separation. Logical tiers keep responsibilities
  testable and allow deployment topology to be chosen later.
- **Alternatives considered**: Microservices were rejected as premature
  operational complexity. A single undifferentiated MVC application was
  rejected because it would mix presentation, policies, persistence, and
  provider behavior.

### Checkout with durable idempotency and explicit pending outcomes

- **Decision**: Create a durable pending checkout/order attempt with a
  customer-scoped unique idempotency key before contacting the payment
  provider. Revalidate totals and stock, reserve all requested units
  atomically, and send a stable provider idempotency key. Confirm the same
  order only after definitive approval. On a definite decline, retain the cart
  and release stock. On timeout or processing, keep the attempt pending and
  reconcile using the same provider reference/key; do not treat ambiguity as
  decline or start a second charge.
- **Rationale**: A local unique checkout key protects the bookstore from
  duplicate orders even if provider key retention is limited. A durable
  pending state allows retries and asynchronous outcomes to resolve against
  the original order. Separating local transaction work from the provider call
  avoids holding a database transaction open over network latency.
- **Alternatives considered**: Calling payment before creating a durable
  checkout record leaves retries and callbacks without a stable local owner.
  Releasing stock on any timeout risks confirming a late payment without
  fulfillable inventory. Blocking synchronously until a final result is
  unsuitable for payment methods with delayed outcomes.

### Inventory represented as on-hand, reserved, and derived available stock

- **Decision**: Maintain `onHand` and `reserved` values and derive available
  stock as `onHand - reserved`. Checkout reservations and staff adjustments
  must atomically validate against the same inventory state. A staff
  adjustment cannot reduce on-hand below reserved quantity; an inventory
  conflict fails without partial changes.
- **Rationale**: Reserving before external payment prevents concurrent
  checkouts from both purchasing the final unit. Recording adjustments with
  actor, reason, before/after values, and time provides an audit trail.
- **Alternatives considered**: Checking stock without reserving it allows
  overselling. Letting staff overwrite a quantity without checking active
  reservations can make pending paid orders unfulfillable. Row locks and
  optimistic version checks are implementation alternatives; neither is
  mandated here.

### Provider adapters with narrow application contracts

- **Decision**: Put payment and fulfillment behind application-owned
  contracts, with adapters translating provider-specific requests, events, and
  failures into bookstore outcomes. Store only provider references and
  tokenized payment references, never raw payment credentials.
- **Rationale**: The feature specification does not select external providers.
  Narrow contracts keep their details out of the domain and make provider
  replacement and integration testing possible.
- **Alternatives considered**: Calling provider SDKs from controllers or
  domain objects couples policies to a vendor and scatters error handling.

### Technology-neutral baseline

- **Decision**: Defer language, framework, storage product, hosting, identity,
  payment, fulfillment, tax, and currency selection.
- **Rationale**: The workspace contains specification documents but no
  application code or platform constraints. Choosing products now would be
  unsupported by the requested baseline.
- **Alternatives considered**: Selecting a popular stack or provider without
  user/project evidence would add arbitrary constraints rather than resolve a
  requirement.

## References

- Microsoft, [N-tier Architecture Style](https://learn.microsoft.com/en-us/azure/architecture/guide/architecture-styles/n-tier)
  — logical layers and physical tiers are distinct; a three-tier arrangement
  commonly separates presentation, middle, and data responsibilities.
- Microsoft, [MVC pattern overview](https://learn.microsoft.com/en-us/aspnet/core/mvc/overview?view=aspnetcore-9.0)
  — controller, view, and model presentation responsibilities.
- OWASP, [Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)
  — least privilege, deny-by-default, and resource-level authorization.
- Stripe, [PaymentIntent lifecycle](https://docs.stripe.com/payments/paymentintents/lifecycle)
  — payment processing can be asynchronous; an intermediate outcome is not
  final approval.
- Stripe, [Webhooks](https://docs.stripe.com/webhooks) — asynchronous provider
  event delivery patterns.
- Stripe, [Idempotent requests](https://docs.stripe.com/api/idempotent_requests)
  — retry semantics and provider-key constraints.
- PostgreSQL, [Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html)
  and [Explicit Locking](https://www.postgresql.org/docs/current/explicit-locking.html)
  — examples of concurrency mechanisms, not a recommendation to use a
  particular database.
