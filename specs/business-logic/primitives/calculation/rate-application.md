# Rate Application

Multiplies a base amount by a rate to produce a derived charge, fee, or discount. Rate application captures the canonical pattern of percentage-based business math — late fees, commissions, taxes, and markup — in one primitive that makes the rate, base, and result explicit and auditable.

## Shape

```yaml
primitive: rate_application
base: invoice.balance
rate: 0.02
target: late_fee
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `base` | field path or number | yes | The amount the rate is applied against |
| `rate` | decimal or field path | yes | The multiplier expressed as a decimal (e.g. `0.02` for 2%); may reference a field resolved at runtime |
| `target` | field path | yes | The field that receives the result of `base × rate` |

## Use when

- A charge or credit is defined as a percentage of another amount (late fees, sales commissions, markup)
- The rate may vary by context and you want to keep the base-times-rate pattern in one place
- You need a traceable link between the base amount, the rate source, and the resulting figure
- The result will be further constrained by a `bound` or adjusted by `rounding`

## Composes with

- [table-lookup](./table-lookup.md) — resolve the rate dynamically from a tax or fee schedule
- [tier-lookup](./tier-lookup.md) — select the rate from a tiered pricing or commission schedule
- [bound](./bound.md) — cap or floor the resulting fee amount
- [rounding](./rounding.md) — round the product to the required currency precision
- [formula](./formula.md) — the base amount may itself be a formula-derived value
- [condition/threshold](../condition/threshold.md) — gate rate application on a threshold being met

## Example

```yaml
primitive: rate_application
base: invoice.balance
rate: 0.02
target: late_fee
```

```yaml
primitive: rate_application
base: order.subtotal
rate: sales_rep.commission_rate
target: commission.amount
```
