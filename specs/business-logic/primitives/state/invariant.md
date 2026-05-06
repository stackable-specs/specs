# Invariant

A constraint that must always hold true for a business entity, regardless of how it arrived at its current state. Invariants express system-wide consistency rules that no transition, effect, or external write may violate.

## Shape

```yaml
primitive: invariant
name: paid_invoice_balance_zero
must_hold:
  - invoice.status = Paid implies invoice.balance = 0
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Unique snake_case identifier for this invariant |
| `must_hold` | list of predicates | yes | One or more logical statements that must evaluate to true at all times |

## Use when

- A relationship between two or more fields must never be violated regardless of the operation performed
- State transitions carry implicit consistency obligations that must be enforced after every change
- Multiple code paths can modify an entity and you need a single authoritative guard
- Regulatory or audit requirements demand that certain conditions are demonstrably always true
- You want to express a business rule that cannot be satisfied by a single transition guard alone

## Composes with

- [state](./state.md) — invariants frequently reference the entity's current state value as part of the predicate
- [transition](./transition.md) — invariants are evaluated after every transition to confirm the resulting state is consistent
- [fact](../fact/fact.md) — invariant predicates reference facts as the source of field values
- [condition](../condition/condition.md) — a condition can re-use an invariant predicate to gate downstream logic
- [terminal-state](./terminal-state.md) — terminal states often carry the strongest invariants, since no future transition can repair a violation

## Example

```yaml
primitive: invariant
name: paid_invoice_balance_zero
must_hold:
  - invoice.status = Paid implies invoice.balance = 0
```

```yaml
primitive: invariant
name: cancelled_order_no_open_shipments
must_hold:
  - order.status = Cancelled implies order.shipments.all(s => s.status = Cancelled or s.status = Returned)
```
