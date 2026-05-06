# Membership

A set-inclusion test that asserts whether a field value belongs to a defined list of allowed or disallowed values. Membership keeps enumerated eligibility sets explicit and centrally declared rather than scattered across comparison chains.

## Shape

```yaml
primitive: membership
value: customer.tier
set:
  - Enterprise
  - Strategic
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `value` | field path | yes | The field whose value is tested for membership |
| `set` | list of literals | yes | The set of values the field is compared against |
| `operator` | enum | no | `in` (default) or `not_in` — whether inclusion or exclusion is asserted |

## Use when

- Eligibility depends on a field matching one of a known list of categorical values (e.g. tier, region, product type)
- You need to exclude a field value from a set (e.g. orders not in a list of blocked countries)
- An enumerated allowlist or denylist must be auditable and changeable without modifying rule expressions
- The same set appears in multiple rules and should be defined once and referenced
- You are routing, filtering, or segmenting based on discrete categorical attributes

## Composes with

- [comparison](./comparison.md) — membership is a semantic specialization of the `in` and `not_in` operators
- [logical-composition](./logical-composition.md) — membership results combine with other conditions in compound rules
- [predicate](./predicate.md) — a membership check can be named as a predicate for reuse (e.g. `customer_is_priority_tier`)
- [fact](../fact/fact.md) — facts source the field value being tested for membership

## Example

```yaml
primitive: membership
value: customer.tier
set:
  - Enterprise
  - Strategic
```

```yaml
primitive: membership
value: order.shipping_country
operator: not_in
set:
  - RU
  - BY
  - KP
```
