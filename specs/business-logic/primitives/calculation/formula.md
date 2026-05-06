# Formula

A mathematical expression that derives a new value from one or more existing field values. Formulas encode the arithmetic core of business rules — subtotals, margins, fees, and other computed figures — in a single, inspectable expression rather than scattered procedural code.

## Shape

```yaml
primitive: formula
target: order.subtotal
expression: sum(line_items.quantity * line_items.unit_price)
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `target` | field path | yes | The field that receives the computed result |
| `expression` | expression string | yes | Mathematical expression using field paths, operators, and functions (`sum`, `min`, `max`, `round`, `abs`, `if`) |

## Use when

- You need to compute a derived monetary value such as a subtotal, margin, or line-item extension
- The calculation depends on more than one field and would otherwise live in multiple places
- You want the derivation to be auditable and testable independent of the surrounding workflow
- A downstream primitive such as `rate_application` or `bound` needs a well-defined numeric input

## Composes with

- [rate-application](./rate-application.md) — apply a percentage to a formula-derived base amount
- [bound](./bound.md) — clamp the formula result to a min/max range
- [rounding](./rounding.md) — round the expression result to a required precision
- [fact](../fact/fact.md) — field values referenced in the expression are often sourced as facts
- [aggregation](./aggregation.md) — aggregation results can feed into a formula as inputs

## Example

```yaml
primitive: formula
target: order.subtotal
expression: sum(line_items.quantity * line_items.unit_price)
```

```yaml
primitive: formula
target: invoice.early_payment_discount
expression: invoice.balance * 0.02 * if(days_until_due <= 10, 1, 0)
```
