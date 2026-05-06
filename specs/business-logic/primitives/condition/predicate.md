# Predicate

A named, reusable boolean expression that evaluates a condition against a business object. Predicates give a logical test a stable identity so it can be referenced by name in rules, decisions, and compositions without repeating the underlying expression.

## Shape

```yaml
primitive: predicate
name: order_is_delivered
expression: order.status = Delivered
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Snake-case identifier used to reference this predicate elsewhere |
| `expression` | string | yes | Boolean expression evaluated against the target entity's fields |

## Use when

- You want to name a condition so it can be referenced across multiple rules without repeating the logic
- The same boolean test appears in more than one place and drift between copies would cause bugs
- You need a human-readable label that communicates intent (e.g. `customer_is_eligible`) rather than a raw expression
- You are composing conditions with `logical_composition` and want each branch to be self-documenting
- You are testing rule coverage and need individually addressable conditions

## Composes with

- [logical-composition](./logical-composition.md) — predicates are the leaf conditions inside `all`, `any`, and `none` operators
- [comparison](./comparison.md) — a comparison can be wrapped in a predicate to give it a reusable name
- [temporal-predicate](./temporal-predicate.md) — time-bound expressions can be named as predicates for reuse in decision trees
- [fact](../fact/fact.md) — facts supply the field values that predicate expressions evaluate

## Example

```yaml
primitive: predicate
name: order_is_delivered
expression: order.status = Delivered
```

```yaml
primitive: predicate
name: subscription_is_active
expression: subscription.status = Active
```
