# Transition

A named, guarded movement of an entity from one state value to another. A transition names the event that triggers it, the states it connects, and any preconditions that must be satisfied before the movement is allowed.

## Shape

```yaml
primitive: transition
entity: invoice
from: Draft
to: Submitted
on: InvoiceSubmitted
preconditions:
  - invoice.total_amount > 0
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `entity` | string | yes | The business entity whose state changes |
| `from` | string | yes | The state value the entity must currently be in |
| `to` | string | yes | The state value the entity will move to on success |
| `on` | event name | yes | The event that initiates the transition attempt |
| `preconditions` | list of predicates | no | All conditions that must be true before the transition is allowed |

## Use when

- An entity moves between two named state values in response to a business event
- Business rules must be validated before the state change is committed
- You need an explicit record of which events are allowed in which states
- The transition itself should be observable (logged, audited, or trigger downstream effects)
- Multiple events can move an entity out of the same state and each path has different guards

## Composes with

- [state](./state.md) — `from` and `to` values must both be members of the entity's state value set
- [invariant](./invariant.md) — invariants are checked after the transition to ensure system consistency
- [terminal-state](./terminal-state.md) — a transition whose `to` value is terminal has no further outgoing transitions
- [event](../event/event.md) — the `on` field references the event that fires the transition
- [condition](../condition/condition.md) — preconditions are evaluated as conditions before the transition executes
- [effect](../effect/effect.md) — side effects triggered immediately after a successful transition

## Example

```yaml
primitive: transition
entity: invoice
from: Draft
to: Submitted
on: InvoiceSubmitted
preconditions:
  - invoice.total_amount > 0
  - invoice.line_items.count > 0
```

```yaml
primitive: transition
entity: invoice
from: Submitted
to: Paid
on: PaymentReceived
preconditions:
  - payment.amount >= invoice.balance
```
