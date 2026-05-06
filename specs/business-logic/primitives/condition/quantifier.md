# Quantifier

A condition that evaluates a predicate across every element of a collection and returns a single truth value based on how many elements satisfy it. Quantifiers bring collection-level logic into rule conditions without requiring loops or aggregation steps outside the rule itself.

## Shape

```yaml
primitive: quantifier
collection: invoice.line_items
operator: all
condition: line_item.tax_code exists
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `collection` | field path | yes | The collection of items to iterate over |
| `operator` | enum | yes | How the per-element results are combined into a single boolean (see operator list below) |
| `condition` | expression or predicate name | yes | The condition evaluated against each element of the collection |
| `n` | integer | conditional | Required when operator is `at_least`, `at_most`, or `exactly` — the numeric bound |

**Operators:** `all`, `any`, `none`, `count`, `at_least`, `at_most`, `exactly`

## Use when

- A rule must hold for every item in a collection (e.g. all line items have a tax code)
- A rule is satisfied if at least one item in a collection meets a condition
- You need to assert that no items in a collection match a disqualifying state
- A numeric threshold applies to the count of qualifying items (e.g. at least 3 active licenses)
- You want to validate collection integrity without writing aggregation logic in an effect

## Composes with

- [logical-composition](./logical-composition.md) — quantifier results compose as branches within `all`, `any`, or `none`
- [predicate](./predicate.md) — named predicates can be used as the per-element `condition`
- [comparison](./comparison.md) — inline comparisons serve as the `condition` when a named predicate is unnecessary
- [threshold](./threshold.md) — `count` operator output can feed into a threshold check
- [fact](../fact/fact.md) — facts sourced from collection elements supply the fields the condition evaluates

## Example

```yaml
primitive: quantifier
collection: invoice.line_items
operator: all
condition: line_item.tax_code exists
```

```yaml
primitive: quantifier
collection: subscription.seats
operator: at_least
n: 1
condition: seat.status = Active
```
