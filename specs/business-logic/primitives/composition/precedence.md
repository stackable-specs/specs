# Precedence

Defines which rule wins when multiple rules produce conflicting or overlapping outcomes. Precedence does not prevent multiple rules from firing — it declares, after they have fired, which one's outcome is authoritative. It separates the "which rules apply?" question from the "which result counts?" question.

## Shape

```yaml
primitive: precedence
strategy: explicit_priority
rules:
  - rule: contract_price
    priority: 100
  - rule: promo_price
    priority: 50
  - rule: list_price
    priority: 10
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `strategy` | enum | yes | Algorithm used to select the winning rule when outcomes conflict |
| `rules` | list of objects | yes (for `explicit_priority`) | Ordered list of rule references with their associated metadata |
| `rules[].rule` | string | yes | Name of the rule |
| `rules[].priority` | integer | yes (for `explicit_priority`) | Numeric priority; higher number wins |
| `scope` | string | no | Limits precedence resolution to a named field or output dimension |

### `strategy` values

| Strategy | Meaning |
|----------|---------|
| `explicit_priority` | The rule with the highest declared priority number wins |
| `most_specific` | The rule with the most conditions (narrowest match) wins |
| `most_recent` | The rule that was most recently created or last modified wins |
| `most_restrictive` | When outputs are comparable, the lower/stricter value wins (e.g. smaller discount, shorter deadline) |
| `most_permissive` | When outputs are comparable, the higher/looser value wins (e.g. larger entitlement, longer grace period) |

## Use when

- Multiple rules can produce a value for the same output field and only one value should apply (pricing, discount rate, approval threshold)
- You are building a layered pricing model where contract prices override promotional prices which override list prices
- Different teams own rules that can overlap and you need a declared arbitration policy rather than an implicit one determined by evaluation order
- A [rule-set](./rule-set.md) uses `first_match` or `priority_order` mode and you need the priority scores to live outside the individual rules
- You want the precedence logic to be auditable independently from the rules it arbitrates

## Composes with

- [rule](./rule.md) — precedence operates over a set of rules whose outcomes are in conflict
- [rule-set](./rule-set.md) — a rule set with `priority_order` evaluation mode delegates ordering to a precedence declaration
- [conflict-resolver](./conflict-resolver.md) — conflict resolution handles structural contradictions; precedence handles cases where multiple valid outcomes must be ranked
- [decision-table](./decision-table.md) — when the decision table `evaluation_mode` is `first_match`, row order encodes precedence implicitly; a precedence primitive makes it explicit

## Example

```yaml
primitive: precedence
strategy: explicit_priority
scope: order.unit_price
rules:
  - rule: contract_price
    priority: 100
  - rule: promo_price
    priority: 50
  - rule: list_price
    priority: 10
```

```yaml
primitive: precedence
strategy: most_specific
scope: shipping.sla_hours
rules:
  - rule: overnight_express_sla
  - rule: regional_standard_sla
  - rule: global_default_sla
```
