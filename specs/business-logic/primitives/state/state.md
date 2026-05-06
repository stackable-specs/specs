# State

A named set of discrete values that a business entity can occupy at any point in time. State gives an entity a lifecycle — it determines what actions are permitted, what rules apply, and what transitions are available.

## Shape

```yaml
primitive: state
entity: invoice
field: invoice.status
values:
  - Draft
  - Submitted
  - Paid
  - Cancelled
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `entity` | string | yes | The business entity that carries this state |
| `field` | field path | yes | The field on the entity that holds the current state value |
| `values` | list of strings | yes | Exhaustive list of all valid state values |

## Use when

- A business entity has a lifecycle with distinct named stages (Draft, Active, Closed)
- Rules, transitions, or effects need to branch on what stage an entity is currently in
- You need to restrict which operations are permitted based on the entity's current condition
- An audit trail requires knowing the prior and current state at every change
- The set of valid values is fixed and must be enforced at the system boundary

## Composes with

- [transition](./transition.md) — each transition moves the entity from one state value to another
- [terminal-state](./terminal-state.md) — marks which state values are exit points with no further transitions
- [hold](./hold.md) — a hold may be conditional on the entity being in a particular state
- [invariant](./invariant.md) — invariants often assert constraints that must hold within a given state
- [condition](../condition/condition.md) — conditions evaluate the current state value to gate downstream logic
- [event](../event/event.md) — events are the triggers that drive state changes

## Example

```yaml
primitive: state
entity: invoice
field: invoice.status
values:
  - Draft
  - Submitted
  - Paid
  - Cancelled
```

```yaml
primitive: state
entity: subscription
field: subscription.status
values:
  - Trialing
  - Active
  - PastDue
  - Cancelled
  - Expired
```
