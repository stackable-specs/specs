# Rounding

Adjusts a numeric value to a specified decimal precision using a declared rounding mode. Rounding is a distinct primitive because the choice of mode — nearest, up, down, or banker's — is a business policy that affects tax compliance, invoice accuracy, and financial reconciliation and must not be left implicit.

## Shape

```yaml
primitive: rounding
value: tax_amount
mode: nearest
precision: 2
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `value` | field path or number | yes | The numeric value to round |
| `mode` | enum | yes | Rounding mode: `nearest`, `up`, `down`, `bankers` |
| `precision` | integer | yes | Number of decimal places to round to; use `0` for whole numbers |
| `target` | field path | no | If provided, writes the rounded result to this field rather than replacing the input |

**Mode definitions:**
- `nearest` — round half up (standard commercial rounding, 2.5 → 3)
- `up` — always round away from zero (ceiling), used when undercharging is never acceptable
- `down` — always round toward zero (floor), used for tax calculations that must not over-collect
- `bankers` — round half to even (IEEE 754), reduces cumulative rounding bias across large datasets

## Use when

- A monetary calculation produces more decimal places than the target currency supports
- Tax, regulatory, or accounting standards mandate a specific rounding convention
- You are aggregating many rounded line items and need to control for cumulative drift
- A `formula` or `rate_application` result must be stored at a defined precision before further use

## Composes with

- [formula](./formula.md) — round the output of a computed expression to currency precision
- [rate-application](./rate-application.md) — round the fee or charge produced by a rate multiplication
- [proration](./proration.md) — round the prorated amount after the period ratio is applied
- [allocation](./allocation.md) — round individual allocation amounts and attribute any penny remainder
- [bound](./bound.md) — apply bound before rounding to avoid limit violations from rounding up

## Example

```yaml
primitive: rounding
value: tax_amount
mode: nearest
precision: 2
```

```yaml
primitive: rounding
value: prorated_subscription_fee
mode: up
precision: 2
target: invoice_line.amount
```
