# Bound

Clamps a computed value to a declared minimum and/or maximum. Bound enforces the guardrails that business rules impose on calculated figures — discount caps, minimum charges, maximum penalties — without embedding those limits inside the expression that produced the value.

## Shape

```yaml
primitive: bound
value: raw_discount
min: 0
max: 250
target: discount.amount
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `value` | field path or number | yes | The input value to clamp |
| `min` | number or field path | no | Inclusive lower bound; the result will never be less than this value |
| `max` | number or field path | no | Inclusive upper bound; the result will never exceed this value |
| `target` | field path | yes | The field that receives the clamped result |

At least one of `min` or `max` must be provided.

## Use when

- A calculated fee, discount, or penalty must not fall below zero or exceed a contractual cap
- Regulatory or policy rules impose an absolute floor or ceiling on a charge
- You want to separate the derivation logic from the limit policy so each can change independently
- A `formula` or `rate_application` result needs safety rails before it is written to a record

## Composes with

- [formula](./formula.md) — bound the output of an arithmetic expression
- [rate-application](./rate-application.md) — cap the fee or discount produced by a rate multiplication
- [proration](./proration.md) — ensure a prorated amount does not exceed the full period charge
- [rounding](./rounding.md) — round after bounding to maintain correct currency precision
- [condition/threshold](../condition/threshold.md) — threshold checks often mirror bound limits; keep them in sync

## Example

```yaml
primitive: bound
value: raw_discount
min: 0
max: 250
target: discount.amount
```

```yaml
primitive: bound
value: computed_late_fee
min: 5.00
max: account.credit_limit
target: late_fee.amount
```
