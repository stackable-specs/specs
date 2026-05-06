# Resource

Something finite that can be owned, reserved, consumed, granted, or accessed. Resources differ from entities in that they are *scarce* — rules around resources control whether they can be allocated, and what happens when they run out.

## Shape

```yaml
primitive: resource
name: <string>
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Name of the scarce thing being managed (e.g. `InventoryItem`, `SupportSeat`, `APIQuota`) |

## Use when

- Rules need to check whether something is available before allowing an action
- An operation consumes a finite quantity (inventory units, license seats, API calls, budget)
- Reservations, holds, or grants need to track what has been committed vs. what remains
- Releasing a resource back to the pool is a possible outcome

## Composes with

- [reserve-resource](../effect/reserve-resource.md) — commits a quantity of this resource
- [release-resource](../effect/release-resource.md) — returns committed quantity to the pool
- [grant-entitlement](../effect/grant-entitlement.md) — grants access to a resource-backed entitlement
- [quota-consumption](../effect/create-obligation.md) — tracks consumption against a limit
- [threshold](../condition/threshold.md) — checks remaining quantity against a minimum

## Example

```yaml
primitive: resource
name: InventoryItem
```

```yaml
primitive: resource
name: APIQuota
```
