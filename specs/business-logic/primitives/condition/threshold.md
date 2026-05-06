# Threshold

A numeric limit check that asserts whether a value exceeds, meets, or falls below a defined boundary. Threshold makes the intent of a numeric gate explicit — distinguishing a policy boundary from a generic comparison — and keeps limit values declarative and auditable.

## Shape

```yaml
primitive: threshold
value: refund.amount
operator: ">"
threshold: 500
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `value` | field path or expression | yes | The numeric value being tested |
| `operator` | enum | yes | The relational operator (`>`, `>=`, `<`, `<=`, `=`, `!=`) |
| `threshold` | number or field path | yes | The boundary value the `value` is compared against |

## Use when

- A rule enforces a policy limit such as a maximum refund amount or minimum order value
- Approval workflows are triggered when a numeric field crosses a defined boundary
- You want to make policy limits visible as named, auditable declarations rather than inline magic numbers
- A compliance rule specifies a numeric cap that may need to be updated independently of the rule logic
- You are distinguishing a business limit from a general-purpose comparison for clarity in rule catalogs

## Composes with

- [comparison](./comparison.md) — threshold is a semantic specialization of comparison; they share operator vocabulary
- [logical-composition](./logical-composition.md) — threshold results combine with other conditions via `all` or `any`
- [predicate](./predicate.md) — a threshold can be wrapped in a predicate to give it a reusable name
- [quantifier](./quantifier.md) — a `count` result from a quantifier can be the `value` in a threshold check
- [fact](../fact/fact.md) — facts source the numeric field values that threshold evaluates

## Example

```yaml
primitive: threshold
value: refund.amount
operator: ">"
threshold: 500
```

```yaml
primitive: threshold
value: order.total_discount_percent
operator: ">="
threshold: 20
```
