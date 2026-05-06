# Rule

The atomic unit of business logic. A rule evaluates a trigger and optional conditions, then produces one or more effects. All other composition primitives orchestrate, combine, or arbitrate between rules — a rule is the thing they operate on.

## Shape

```yaml
primitive: rule
when:
  trigger: PaymentFailed
if:
  - invoice.amount_due > 0
then:
  - create_obligation: CollectionFollowUpTask
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `when` | object | yes | The trigger that activates this rule |
| `when.trigger` | string | yes | Event or state change that fires the rule |
| `if` | list of predicates | no | Conditions that must hold for the `then` block to execute |
| `then` | list of effects | yes | Effects produced when the rule fires and all conditions pass |

## Use when

- You have a discrete business decision that maps one or more conditions to a specific outcome
- You need a reusable unit of logic that can be collected into a [rule-set](./rule-set.md) or routed by a [branch](./branch.md)
- The logic encodes a policy, pricing decision, approval requirement, or entitlement check
- You want a single, auditable record of a business decision that can be versioned independently
- Multiple downstream systems need to react to the same business event

## Composes with

- [rule-set](./rule-set.md) — rules are grouped and evaluated together inside a rule set
- [condition](../condition/condition.md) — the `if` block is made up of one or more conditions
- [event](../event/event.md) — the `when.trigger` references a named event
- [effect](../effect/effect.md) — the `then` block produces one or more typed effects
- [precedence](./precedence.md) — when multiple rules conflict, a precedence primitive decides which rule wins
- [conflict-resolver](./conflict-resolver.md) — detects and resolves contradictions between this rule and others

## Example

```yaml
primitive: rule
when:
  trigger: PaymentFailed
if:
  - invoice.amount_due > 0
then:
  - create_obligation: CollectionFollowUpTask
```

```yaml
primitive: rule
when:
  trigger: OrderPlaced
if:
  - order.total > 1000
  - customer.tier = Enterprise
then:
  - assign_approval: SeniorSalesReview
  - set_flag: high_value_order
```
