# Reserve Resource

Commits a quantity of a named resource, removing it from the available pool and preventing it from being allocated to any other consumer. The reservation is transient by default — it carries an expiry, after which the quantity is automatically released if it has not been converted into a confirmed allocation. Use it to implement soft-hold semantics before a hard commitment is made.

## Shape

```yaml
primitive: reserve_resource
resource: inventory.sku
quantity: order_line.quantity
expires_at: now + 15m
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `resource` | field path or string | yes | Dot-path identifying the resource being reserved (e.g., `inventory.sku`, `seat_pool.conference_id`, `credit_limit.account_id`) |
| `quantity` | number or field path | yes | The amount to hold; may be a literal or a field path resolving to a numeric value |
| `expires_at` | timestamp or expression | yes | When the reservation lapses if not confirmed; prevents perpetual holds that drain the available pool |
| `reservation_id_field` | field path | no | Field on the requesting entity where the generated reservation ID will be written, enabling later `release_resource` calls |
| `on_insufficient` | enum | no | Behavior when the requested quantity is unavailable: `block` (default, prevents reservation), `partial` (reserves what is available), `queue` (waits for availability) |

## Use when

- A checkout or ordering flow must hold inventory for a user while they complete payment, preventing oversell
- A seat, slot, or capacity unit must be locked for a specific consumer during a multi-step workflow
- A financial credit line or budget must be soft-committed before a transaction is confirmed
- You need optimistic allocation with an automatic rollback path (expiry) if the downstream confirmation never arrives
- Multiple concurrent requests compete for the same limited resource and must be serialized without a distributed lock

## Composes with

- [release-resource](./release-resource.md) — the mirror effect; releases the held quantity back to the pool when the reservation is confirmed or abandoned
- [entity](../entity/resource.md) — the resource being reserved should be declared as a resource entity so capacity tracking is explicit
- [reserve-resource](./reserve-resource.md) — multiple reservations on the same resource accumulate; the pool decrements for each active hold
- [block-action](./block-action.md) — if the reservation fails due to insufficient quantity, emit a `block_action` to prevent the order or booking from proceeding
- [event](../event/event.md) — emit a `ResourceReserved` event so downstream fulfillment systems can react to the hold immediately

## Example

```yaml
primitive: reserve_resource
resource: inventory.sku
quantity: order_line.quantity
expires_at: now + 15m
reservation_id_field: order_line.reservation_id
on_insufficient: block
```

```yaml
primitive: reserve_resource
resource: conference.seat_pool
quantity: 1
expires_at: now + 30m
reservation_id_field: registration.seat_reservation_id
on_insufficient: queue
```
