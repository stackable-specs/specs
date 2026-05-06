# Logical Composition

A boolean operator applied to a set of conditions to produce a single truth value. Logical composition is how individual predicates and comparisons are combined into compound eligibility tests, gate checks, and multi-clause rules.

## Shape

```yaml
primitive: logical_composition
operator: all
conditions:
  - order.status = Delivered
  - days_since(order.delivered_at) <= 30
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `operator` | enum | yes | The logical operator governing how conditions are combined (see operator list below) |
| `conditions` | list of expressions or predicate names | yes | The conditions to evaluate; each item may be an inline expression or a named predicate reference |

**Operators:** `all` (AND), `any` (OR), `none` (NOR), `not` (unary negation), `xor` (exactly one true)

## Use when

- A business rule requires multiple conditions to all be satisfied simultaneously
- Eligibility has alternative paths where satisfying any one condition is sufficient
- You need to express exclusion logic (none of the following must be true)
- A condition must be negated without introducing a separate predicate definition
- You are building a decision tree branch that depends on mutually exclusive conditions via `xor`

## Composes with

- [predicate](./predicate.md) — named predicates are the preferred input to `conditions` for readability and reuse
- [comparison](./comparison.md) — comparisons can be inlined directly as condition list items
- [quantifier](./quantifier.md) — quantifier results (e.g. all line items taxed) can be conditions in a composition
- [temporal-predicate](./temporal-predicate.md) — time-bound expressions compose naturally as branches in `all` or `any`
- [threshold](./threshold.md) — threshold results slot in as conditions alongside other checks

## Example

```yaml
primitive: logical_composition
operator: all
conditions:
  - order.status = Delivered
  - days_since(order.delivered_at) <= 30
```

```yaml
primitive: logical_composition
operator: any
conditions:
  - customer.tier = Enterprise
  - customer.annual_spend >= 50000
  - account.contract_type = Strategic
```
