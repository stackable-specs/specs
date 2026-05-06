# Derived Fact

A fact computed from other facts rather than sourced directly from a system. Derived facts name intermediate calculations that multiple rules share, preventing logic duplication and making computations auditable.

## Shape

```yaml
primitive: derived_fact
name: <field_path>
derive:
  expression: <expression>
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | field path | yes | Dot-notation name for the computed value (e.g. `invoice.days_past_due`) |
| `derive.expression` | expression | yes | The formula or function that produces the value from other facts |

## Use when

- Multiple rules need the same computed value — define it once as a derived fact rather than repeating the expression
- A calculation is complex enough to name (e.g. `days_since_delivery` is clearer than repeating `days_between(order.delivered_at, today)` everywhere)
- You want the intermediate value to appear in audit captures and decision traces

## Composes with

- [fact](./fact.md) — derived facts are built from raw facts
- [fact-snapshot](./fact-snapshot.md) — snapshots should capture derived facts alongside source facts for replay
- [predicate](../condition/predicate.md) — derived facts are frequently the subject of predicates
- [formula](../calculation/formula.md) — a formula can itself be the source of a derived fact

## Example

```yaml
primitive: derived_fact
name: invoice.days_past_due
derive:
  expression: days_between(invoice.due_date, today)
```

```yaml
primitive: derived_fact
name: order.item_count
derive:
  expression: count(order.line_items)
```
