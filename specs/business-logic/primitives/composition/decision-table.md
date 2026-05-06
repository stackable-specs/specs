# Decision Table

Condition/outcome pairs expressed in tabular form. Useful when many combinations of inputs map to specific outputs and expressing that as prose rules would be repetitive or error-prone. Each row is an independent when/then pair; the table is evaluated top-to-bottom and the first matching row wins unless overridden by an explicit evaluation mode.

## Shape

```yaml
primitive: decision_table
inputs:
  - customer.tier
  - order.amount
rows:
  - when:
      customer.tier: Enterprise
      order.amount: "> 10000"
    then:
      review_required: true
  - when:
      customer.tier: SMB
      order.amount: "> 50000"
    then:
      review_required: true
default:
  review_required: false
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `inputs` | list of field paths | yes | The fields whose values are matched against each row |
| `rows` | list of objects | yes | Ordered list of when/then pairs |
| `rows[].when` | map | yes | Conditions keyed by input field; supports literals and comparison expressions |
| `rows[].then` | map | yes | Output values to apply when this row matches |
| `default` | map | no | Output values to apply when no row matches; required when the input space is unbounded |
| `evaluation_mode` | enum | no | `first_match` (default) or `collect_all` |

## Use when

- A policy maps many discrete combinations of two or more inputs to different outputs
- The same inputs appear repeatedly across many rules, making a table easier to read and maintain than individual rule files
- Non-engineers need to review or edit the logic — a table is the most legible representation for that audience
- You are modelling approval thresholds, pricing tiers, discount schedules, or SLA targets that vary by multiple dimensions
- The set of input combinations is finite and enumerable

## Composes with

- [rule](./rule.md) — each table row is semantically equivalent to a rule; a decision table is a compact encoding of a rule set
- [rule-set](./rule-set.md) — a decision table can be included as a member of a larger rule set
- [condition](../condition/condition.md) — row `when` clauses may embed complex conditions rather than simple equality checks
- [fact](../fact/fact.md) — input field paths reference facts sourced from entities
- [effect](../effect/effect.md) — row `then` clauses produce typed effects

## Example

```yaml
primitive: decision_table
name: order_review_requirements
inputs:
  - customer.tier
  - order.amount
rows:
  - when:
      customer.tier: Enterprise
      order.amount: "> 10000"
    then:
      review_required: true
  - when:
      customer.tier: SMB
      order.amount: "> 50000"
    then:
      review_required: true
default:
  review_required: false
```

```yaml
primitive: decision_table
name: shipping_sla_targets
inputs:
  - shipment.priority
  - destination.region
rows:
  - when:
      shipment.priority: Overnight
      destination.region: Domestic
    then:
      sla_hours: 24
  - when:
      shipment.priority: Standard
      destination.region: Domestic
    then:
      sla_hours: 120
  - when:
      shipment.priority: Standard
      destination.region: International
    then:
      sla_hours: 240
default:
  sla_hours: 120
```
