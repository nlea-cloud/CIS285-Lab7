# Fulfillment Provider Contract

This is a provider-neutral application boundary for the fulfillment adapter.
The selected provider's wire format and authentication are implementation
decisions.

## Submit Fulfillment

**Caller**: Fulfillment application use case, only for a confirmed order.

**Request**:

| Field | Meaning |
|---|---|
| `merchantOrderReference` | Stable local order reference |
| `idempotencyKey` | Stable key for submission retries |
| `recipient` | Customer delivery name and contact details required for delivery |
| `deliveryAddress` | Confirmed delivery address snapshot |
| `items` | Book identifiers or fulfillment identifiers and confirmed quantities |
| `requestedAt` | Submission timestamp |

**Outcome**:

- `Accepted(providerShipmentReference, initialStatus)`
- `Rejected(reasonCode)`
- `Pending(correlationReference?)`
- `Unknown(correlationReference?)`

Repeated submission with the same idempotency key MUST resolve to the original
fulfillment request where the provider supports idempotency. A timeout is
unknown, not proof that no shipment request was accepted.

## Receive Shipment Progress

**Input**: verified provider update containing `providerShipmentReference`,
`eventReference`, status, event time, and tracking details when available.

The adapter verifies and correlates the update to a known local shipment,
normalizes provider status into `Processing`, `Shipped`, `Delivered`, or
`Exception`, and forwards a domain update. Duplicate event references are
ignored. An update MUST NOT alter payment state, purchased quantities, order
ownership, or the price snapshots.

## Application Guarantees

- Only confirmed orders are submitted for fulfillment.
- The customer sees the latest known status; absence of a provider update is
  represented as pending/unavailable status information, not as a fabricated
  shipment event.
- Staff-entered progress and provider progress are associated with the same
  order and auditable by source and time.
- Provider-specific credentials, callback verification, retry schedules, and
  status mappings belong in the adapter, not in the domain model.
