# Payment Provider Contract

This is a provider-neutral application boundary, not a prescribed network API.
The payment adapter translates the selected provider's format into these
requests and outcomes.

## Authorize Payment

**Caller**: Checkout application use case.

**Preconditions**:

- The customer has reviewed the current total and delivery details.
- A pending checkout and order exist, and required inventory is reserved.
- The amount and currency are the server-calculated order values.
- Payment details are represented by a provider-managed or tokenized reference;
  raw payment credentials are not persisted by the bookstore.

**Request**:

| Field | Meaning |
|---|---|
| `merchantPaymentReference` | Stable unique reference for this order payment attempt |
| `idempotencyKey` | Stable provider-operation key reused only for retries of the same authorization |
| `amount` | Exact amount to authorize, equal to the order total |
| `currency` | Currency of the amount |
| `paymentMethodToken` | Provider-managed/tokenized payment reference |
| `customerReference` | Non-sensitive customer reference required by the provider |
| `returnContext` | Opaque correlation data; no authorization decision is encoded here |

**Outcome**:

- `Approved(providerReference, authorizedAmount)`
- `Declined(reasonCode)`
- `Pending(providerReference?)`
- `Unknown(correlationReference?)`

An adapter reports a network timeout or unparseable response as `Unknown`, not
as `Declined`. The application verifies the amount and correlation before it
applies an outcome.

## Reconcile Payment

**Caller**: Checkout recovery process or verified provider event handler.

**Request**: `merchantPaymentReference` and the original provider
`idempotencyKey` or `providerReference`.

**Outcome**: `Approved`, `Declined`, `Pending`, `Voided`, or `Unknown`, with
provider reference, amount, and currency when available.

Provider events MUST be authenticated/verified by the adapter, correlated to
the expected local attempt, and safe to process more than once. Out-of-order
or duplicate events MUST NOT regress a final payment outcome.

## Application Guarantees

- The same checkout retry does not create another local payment attempt or
  order; the same provider idempotency key is used for retries of the same
  external operation.
- The order remains unconfirmed for `Pending` or `Unknown`.
- A definite decline is recorded and releases the reservation; the cart remains
  available.
- A late approval after a reservation is no longer valid cannot silently
  confirm an unfulfillable order. The application follows a provider-supported
  void/refund recovery policy.
- Provider-specific key-retention periods, signature verification, retries,
  and reconciliation schedules are adapter configuration/policy decisions.
