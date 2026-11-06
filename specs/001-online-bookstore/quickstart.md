# Online Bookstore Architecture Validation Quickstart

## Scope and Prerequisites

This feature currently defines a base architecture; the repository has no
application source, selected language/framework, or runnable test harness.
Therefore there is no truthful startup or test-run command yet. These scenarios
are executable acceptance checks once an implementation and its run commands
exist. The storage and provider adapters may be replaced by controlled test
doubles for deterministic outcomes.

Read [architecture.md](./architecture.md),
[data-model.md](./data-model.md), and the contracts in
[contracts/](./contracts/) before validating a concrete implementation.

## Validation Scenarios

### Catalog and cart

1. Prepare an active in-stock book, and an inactive book.
2. As a visitor, search by title, author, and ISBN; filter by category and
   availability; open the active book's details.
3. Add a valid quantity to a customer's cart, change it, remove another line,
   and inspect the displayed prices and total.
4. Verify inactive books do not appear in public results and no-match searches
   return an empty result without error.

**Expected**: results and cart totals match the current catalog data, but cart
prices and quantities are not treated as guaranteed at checkout.

### Successful checkout and duplicate submission

1. Prepare a customer cart with available stock and a payment test outcome of
   approved.
2. Submit checkout with delivery details and one stable checkout key.
3. Repeat the same submission, including a concurrent retry if the harness
   supports it.
4. Inspect the order history and inventory.

**Expected**: the customer sees the complete total before payment; one pending
attempt/order is created; one authorization is issued for the stable payment
key; exactly one order is confirmed; reserved stock is consumed once; the
receipt contains purchased item and price snapshots.

### Declined and unknown payment

1. Retry a new checkout using a controlled declined payment outcome.
2. Verify no confirmed order exists and the cart remains available.
3. Start another checkout and simulate a timeout or provider `Pending` outcome.
4. Retry/reconcile with the original provider reference/key, first with a
   pending result and then with a final approved or declined result.

**Expected**: decline is recorded without confirming an order and releases
stock; an unknown outcome remains pending and is not treated as a decline or
approval; retries do not duplicate orders or charges; final reconciliation
updates the original attempt.

### Inventory concurrency and staff adjustment

1. Set one book's available quantity to one.
2. Submit two checkout reservations for one unit at the same time.
3. As authorized staff, attempt a stock adjustment that would set on-hand
   quantity below active reservations, then submit a valid adjustment with a
   reason.
4. Inspect current availability and the inventory adjustment history.

**Expected**: at most one checkout reserves the final unit; the invalid staff
adjustment makes no change and reports the conflict; the valid change updates
availability and records actor, before/after quantities, reason, and time.

### Authorization and fulfillment

1. Sign in as two different customers and create an order for the first.
2. Attempt to access the first customer's order from the second account.
3. As authorized staff, submit a confirmed order to the fulfillment test
   provider and apply a duplicate shipment update.

**Expected**: the second customer cannot see the first customer's order; only
staff can change catalog, inventory, and fulfillment; the provider update
appears once against the correct order; no shipment update is represented
distinctly from a reported shipment status.

## Implementation Run Commands

Not applicable until a technology stack and runnable application are selected.
When they are, add the exact setup, startup, focused test, and full test
commands here; do not replace the expected outcomes above with framework-only
checks.
