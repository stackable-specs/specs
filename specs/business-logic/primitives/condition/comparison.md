# Comparison

A two-operand test that evaluates a relationship between a field value and a literal, another field, or a computed value. Comparisons are the atomic building blocks of numeric, string, and set-based conditions.

## Shape

```yaml
primitive: comparison
left: refund.amount
operator: ">"
right: 500
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `left` | field path or literal | yes | The left-hand operand — typically a field reference on the target entity |
| `operator` | enum | yes | The comparison operation to perform (see operator list below) |
| `right` | field path, literal, or list | yes | The right-hand operand — a literal value, another field, or a list for set operators |

**Operators:** `=`, `!=`, `>`, `>=`, `<`, `<=`, `in`, `not_in`, `contains`, `exists`, `missing`, `matches`

## Use when

- You need to test a field against a threshold, constant, or enumerated value
- You are checking set membership (e.g. customer tier is in a list of qualifying tiers)
- You need to assert presence or absence of a field before acting on its value
- You are validating format or structure with a regex pattern via `matches`
- You want a single, auditable expression rather than embedding logic in prose

## Composes with

- [predicate](./predicate.md) — a comparison can be wrapped in a predicate to give it a reusable name
- [logical-composition](./logical-composition.md) — multiple comparisons are combined with `all`, `any`, or `none`
- [threshold](./threshold.md) — threshold specializes comparison for numeric limit checks with richer semantics
- [membership](./membership.md) — membership specializes comparison for set-based inclusion tests
- [fact](../fact/fact.md) — facts supply the field values referenced in the left and right operands

## Example

```yaml
primitive: comparison
left: refund.amount
operator: ">"
right: 500
```

```yaml
primitive: comparison
left: invoice.days_overdue
operator: ">="
right: 90
```

```yaml
primitive: comparison
left: customer.email
operator: matches
right: ".+@.+\\..+"
```
