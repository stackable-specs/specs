# Release Resource

Returns a previously committed resource quantity to the available pool, making it eligible for allocation by other consumers. Release is the complement to reservation — it fires when a soft hold is either explicitly abandoned (order cancelled, checkout timed out) or superseded by a confirmed hard allocation that no longer needs the transient hold. Without explicit release, pools drain and reservations must wait for expiry.

## Shape

```yaml
primitive: release_resource
reservation_id: reservation.id
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `reservation_id` | field path or literal | yes | Identifier of the reservation to cancel; must reference a reservation previously created by a `reserve_resource` effect |
| `reason_code` | string | no | Machine-readable code explaining why the release was triggered (e.g., `OrderCancelled`, `CheckoutAbandoned`, `PaymentConfirmed`, `ReservationExpired`) |
| `released_by` | string | no | Name of the rule or event that triggered the release, for audit and inventory reconciliation |

## Use when

- An order or booking is cancelled before confirmation and the held inventory or capacity must be returned to the available pool
- A payment succeeds and the soft inventory hold should be converted to a confirmed fulfillment allocation, releasing the transient reservation
- A checkout session times out and the cart's held quantities must be freed for other shoppers
- A multi-step saga rolls back and every resource reserved during earlier steps must be explicitly released
- You need an auditable record of when and why a hold was lifted, separate from the automatic expiry path

## Composes with

- [reserve-resource](./reserve-resource.md) — every `release_resource` must reference a `reservation_id` that was produced by a prior `reserve_resource`
- [entity](../entity/resource.md) — the resource pool entity tracks available quantity by summing active reservations; release decrements that count
- [transition](../state/transition.md) — transitions to cancelled or terminal states (e.g., `Pending` → `Cancelled`) typically emit `release_resource` for each active hold on the entity
- [event](../event/event.md) — an `OrderCancelled` or `PaymentConfirmed` event is the common trigger; the event carries the reservation ID in its payload
- [post-ledger-entry](./post-ledger-entry.md) — when the resource is financial (e.g., a credit reserve), releasing it pairs with a reversal ledger entry

## Example

```yaml
primitive: release_resource
reservation_id: order_line.reservation_id
reason_code: OrderCancelled
released_by: OrderCancellationRule
```

```yaml
primitive: release_resource
reservation_id: registration.seat_reservation_id
reason_code: PaymentConfirmed
released_by: RegistrationConfirmationRule
```
