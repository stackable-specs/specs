# Tier Lookup

Resolves a value by finding the tier whose range contains a given numeric input. Tier lookup captures stepped or banded business rules — usage pricing, tax brackets, volume discounts — where the result changes at defined thresholds rather than varying continuously.

## Shape

```yaml
primitive: tier_lookup
value: usage.units
tiers:
  - from: 0
    to: 1000
    result: 0.10
  - from: 1001
    to: 10000
    result: 0.08
  - from: 10001
    to: null
    result: 0.05
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `value` | field path or number | yes | The numeric input used to select a tier |
| `tiers` | list of tier objects | yes | Ordered list of range-to-result mappings; evaluated top-to-bottom, first match wins |
| `tiers[].from` | number | yes | Inclusive lower bound of the tier range |
| `tiers[].to` | number or null | yes | Inclusive upper bound; `null` means unbounded (open-ended top tier) |
| `tiers[].result` | any | yes | The value returned when the input falls within this tier |

## Use when

- Pricing, rates, or limits change in discrete steps as a numeric value crosses thresholds
- You are modeling usage-based billing, progressive tax brackets, or volume discount bands
- The result is a rate that will feed into a `rate_application` rather than being a direct price
- A `table_lookup` would not work because the key is a continuous range, not a discrete value

## Composes with

- [rate-application](./rate-application.md) — the resolved tier result typically becomes the rate applied to a base amount
- [table-lookup](./table-lookup.md) — use when keys are discrete categorical values rather than ranges
- [aggregation](./aggregation.md) — the aggregated total often serves as the input value for tier selection
- [formula](./formula.md) — formula output can drive the tier lookup input
- [condition/threshold](../condition/threshold.md) — threshold conditions use similar boundary logic but return boolean, not a value

## Example

```yaml
primitive: tier_lookup
value: usage.units
tiers:
  - from: 0
    to: 1000
    result: 0.10
  - from: 1001
    to: 10000
    result: 0.08
  - from: 10001
    to: null
    result: 0.05
```

```yaml
primitive: tier_lookup
value: order.subtotal
tiers:
  - from: 0
    to: 499.99
    result: 0.00
  - from: 500
    to: 1999.99
    result: 0.05
  - from: 2000
    to: null
    result: 0.10
```
