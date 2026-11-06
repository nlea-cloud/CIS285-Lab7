# Online Bookstore Requirements

## 1. Purpose and Scope

This document defines the unified baseline requirements for a single-store
online bookstore. Customers browse and purchase physical books; authorized
staff maintain the catalog, prices, inventory, and fulfillment status. The
architecture is a modular layered application. These requirements specify
observable behavior and quality constraints without choosing a language,
framework, database product, payment provider, or deployment platform.

### 1.1 System Actors

- **Visitor**: Browses, searches, filters, and views the public catalog.
- **Customer**: Creates an account, manages a persistent cart, places orders,
  and views their own order and shipment status.
- **Bookstore staff**: Maintains listings, prices, stock, and order fulfillment.
- **Payment provider**: Authorizes, declines, or reports an unresolved payment
  outcome.
- **Fulfillment provider**: Accepts shipment requests and may report shipment
  progress.

### 1.2 Scope Boundaries and Assumptions

- The initial scope is one bookstore and physical books.
- Browsing does not require an account; placing an order requires an active
  signed-in customer.
- The bookstore defines supported delivery regions, taxes, charges, payment
  methods, and currency. Checkout displays the complete applicable total
  before payment authorization.
- Returns, refunds, promotions, reviews, and digital-book delivery are
  outside this baseline.
- Implementation technologies and numeric service availability/throughput
  targets are not selected here.

## 2. Functional Requirements

### 2.1 Customer Storefront and Cart

- **FR-001**: The system MUST show visitors active book listings with title,
  author, current price, and availability.
- **FR-002**: The system MUST provide full-text search across book titles,
  authors, and descriptions, support ISBN lookup, and allow filtering by
  category and availability.
- **FR-003**: The system MUST allow visitors to open a book detail view with
  its available description, ISBN, category, price, and stock availability.
- **FR-004**: The system MUST return an explicit empty result when no active
  book matches a search or filter.
- **FR-005**: The system MUST allow an active customer to add books to a
  persistent cart, change positive whole-number quantities, and remove lines.
- **FR-006**: The system MUST show each cart line's current displayed price,
  quantity, subtotal, and the cart total.
- **FR-007**: A cart MUST remain available after declined or unresolved
  payment; successful order confirmation MAY clear only the items included in
  that order.
- **FR-008**: The system MUST NOT treat a cart's displayed price or stock as a
  checkout guarantee; checkout revalidates the catalog listing, price, and
  availability.

### 2.2 Accounts and Access

- **FR-009**: The system MUST allow a visitor to create an account and an
  active customer to sign in before placing an order.
- **FR-010**: The system MUST scope cart, account, and order reads to the
  authenticated customer's identity and MUST prevent a customer from viewing
  another customer's data.
- **FR-011**: The system MUST restrict catalog, price, inventory, and
  fulfillment mutations to authorized staff roles.

### 2.3 Checkout, Inventory, and Payment

- **FR-012**: Before accepting checkout, the system MUST recalculate the
  current book prices, taxes, delivery charges, and total, and MUST present
  the complete quote to the customer for acceptance.
- **FR-013**: The system MUST reject checkout when any book is inactive or
  requested quantity is not available, and MUST identify the affected lines
  for customer correction.
- **FR-014**: The system MUST reserve every requested inventory line
  atomically: either all requested quantities are reserved or none are.
- **FR-015**: The system MUST enforce a single checkout attempt per stable,
  customer-scoped idempotency key. A repeat with the same key and request MUST
  return the existing attempt; reuse of the key with a different request MUST
  be rejected. A new customer-approved retry MUST use a new key.
- **FR-016**: The system MUST create one pending-payment order and payment
  attempt for a valid checkout and MUST submit the exact accepted order total
  and currency for authorization.
- **FR-017**: The system MUST use a provider-managed or tokenized payment
  reference and MUST NOT persist raw payment card numbers or security codes.
- **FR-018**: The system MUST confirm an order only after verified payment
  approval matching the order amount and currency.
- **FR-019**: The system MUST record a definitive decline, leave the order
  unconfirmed, release its inventory reservation safely, and retain the cart.
- **FR-020**: For a pending or unknown payment outcome, the system MUST keep
  the payment attempt and order pending, MUST NOT confirm the order, and MUST
  reconcile the original provider attempt before retrying or releasing the
  reservation.
- **FR-021**: The system MUST make reservation release and consumption
  idempotent and MUST NOT release a reservation solely because a local request
  timed out.
- **FR-022**: The system MUST show a successful customer the order reference,
  ordered items, total, and current order status.

### 2.4 Staff Workspace and Fulfillment

- **FR-023**: Authorized staff MUST be able to create, update, activate, and
  deactivate book listings.
- **FR-024**: Authorized staff MUST be able to change book prices. Price
  changes MUST NOT rewrite prices already captured in an order.
- **FR-025**: Authorized staff MUST be able to adjust on-hand inventory only
  when the resulting quantity is nonnegative and not less than active
  reservations.
- **FR-026**: Every staff inventory adjustment MUST record the acting staff
  identity, quantity delta, reason, timestamp, and before/after quantity.
- **FR-027**: Authorized staff MUST be able to review order items, quantities,
  delivery details, and payment outcome needed for fulfillment.
- **FR-028**: The system MUST allow fulfillment progress to be recorded and
  associated with the correct confirmed order.
- **FR-029**: The system MUST allow a customer to view only their own order
  history, including reference, date, total, and latest order/shipment status.
- **FR-030**: The system MUST distinguish a shipment with no available update
  from a shipment with a reported status.
- **FR-031**: Repeated fulfillment requests and duplicate provider updates
  MUST NOT create duplicate shipments or regress a final shipment state.

## 3. Non-Functional Requirements

### 3.1 Modularity and Layer Boundaries

- **NFR-001**: The application MUST maintain explicit presentation,
  application/domain, and data/integration boundaries with zero cross-tier
  boundary violations. Dependencies MUST flow inward toward application and
  domain policies.
- **NFR-002**: Views MUST contain no SQL or persistence access and MUST NOT
  implement business rules. Controllers MUST delegate business operations to
  application use cases.
- **NFR-003**: Persistence and provider adapters MUST implement defined ports
  and MUST NOT calculate bookstore prices, decide order policies, or own
  domain state transitions.
- **NFR-004**: Domain policies and inventory invariants MUST have no dependency
  on presentation, persistence vendor, or provider-specific libraries.

### 3.2 Security and Privacy

- **NFR-005**: The system MUST enforce role-based authorization for staff
  actions and resource ownership for customer data at the application
  boundary. UI hiding or unguessable identifiers MUST NOT be the sole access
  control.
- **NFR-006**: Authenticated identity and role claims MUST be derived from a
  trusted authentication context, not from caller-controlled customer or
  staff identifiers.
- **NFR-007**: The system MUST NOT persist raw payment card numbers or security
  codes. Provider credentials and token references MUST be access-controlled.
- **NFR-008**: Logs and integration messages MUST exclude raw payment
  credentials and minimize personal information to what the receiving
  component needs.

### 3.3 Data Integrity and Resilience

- **NFR-009**: For each book, `available = onHand - reserved`; values MUST
  remain nonnegative and reservation operations MUST be atomic under
  concurrent checkout and staff updates.
- **NFR-010**: A multi-book reservation MUST be all-or-nothing. The system MUST
  prevent overselling even when simultaneous checkouts request the final
  available copies.
- **NFR-011**: Payment states MUST distinguish `Pending`, `Approved`,
  `Declined`, and `Unknown`. `Pending` and `Unknown` MUST NOT be treated as
  `Declined` or `Approved`.
- **NFR-012**: Every externally retried checkout, provider request, reservation
  command, and fulfillment submission MUST have a stable idempotency or
  deduplication reference and a defined duplicate-handling behavior.
- **NFR-013**: Changes that require reliable asynchronous external work MUST
  persist the business state and an outbox message atomically. Consumers MUST
  safely tolerate duplicate delivery.
- **NFR-014**: Order lines MUST preserve a snapshot of purchased book title,
  quantity, price, currency, and line total; subsequent catalog changes MUST
  NOT alter order history.
- **NFR-015**: Staff inventory mutations and payment/order state transitions
  MUST be auditable with actor or correlation references and timestamps.

## 4. Measurable Success Criteria

- **SC-001**: At least 90% of representative test users find a specified
  in-stock book and add it to their cart within two minutes.
- **SC-002**: At least 95% of representative test users complete a valid
  purchase in under five minutes, excluding time spent waiting for an external
  payment decision.
- **SC-003**: In all payment tests, approved payment yields exactly one
  confirmed order; declined payment yields none; pending/unknown payment never
  appears confirmed before reconciliation.
- **SC-004**: Concurrent checkout tests never reserve more units than
  available on-hand stock; failed multi-line reservation leaves all lines
  unchanged.
- **SC-005**: At least 95% of customers in usability tests locate their own
  latest order status within one minute.
- **SC-006**: In access-control tests, zero customers can view another
  customer's account or order details.
- **SC-007**: Every staff stock-adjustment test records actor, delta, reason,
  timestamp, and correct before/after quantities.
