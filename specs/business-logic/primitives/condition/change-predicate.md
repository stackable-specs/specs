# Change Predicate

A condition that evaluates whether a specific field transitioned from one value to another. Change predicates let rules react to state mutations rather than static snapshots — enabling event-driven logic without coupling rules to the change-detection mechanism.

## Shape

```yaml
primitive: change_predicate
field: subscription.status
from: Trial
to: Active
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `field` | field path | yes | The field being observed for a value change |
| `from` | literal or `*` | yes | The value the field held before the change; use `*` to match any prior value |
| `to` | literal or `*` | yes | The value the field holds after the change; use `*` to match any new value |

## Use when

- A rule should fire only when a field moves from a specific prior state to a specific new state, not on every update
- You are implementing event-driven workflows triggered by lifecycle transitions (e.g. trial to paid, pending to approved)
- You need to distinguish a genuine state change from a no-op update where the field value did not actually shift
- Audit or notification logic must capture the exact transition rather than the current value alone
- You are composing a trigger condition for a downstream effect that should not repeat on subsequent saves

## Composes with

- [predicate](./predicate.md) — a change predicate can be named as a predicate so transition conditions are reusable
- [logical-composition](./logical-composition.md) — change predicates combine with other conditions (e.g. a transition AND an account being in a qualifying tier)
- [temporal-predicate](./temporal-predicate.md) — the moment a change occurs is a natural anchor for time-window checks in the same rule
- [membership](./membership.md) — the `to` or `from` value can be generalized to a set using membership when multiple transitions should trigger the same logic
- [fact](../fact/fact.md) — facts supply the before and after field values observed by the change predicate

## Example

```yaml
primitive: change_predicate
field: subscription.status
from: Trial
to: Active
```

```yaml
primitive: change_predicate
field: order.approval_status
from: Pending
to: Approved
```
