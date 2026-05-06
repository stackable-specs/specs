# Pattern Match

A regular-expression test that asserts whether a string field conforms to a defined structural pattern. Pattern match externalizes format validation rules from application code so they are visible, testable, and governed as business logic.

## Shape

```yaml
primitive: pattern_match
field: customer.email
pattern: ".+@.+\\..+"
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `field` | field path | yes | The string field whose value is tested against the pattern |
| `pattern` | string (regex) | yes | A regular expression the field value must match |
| `flags` | string | no | Optional regex flags (e.g. `i` for case-insensitive) |

## Use when

- A field must conform to a structural format before it can be used in downstream logic (e.g. email, phone, postal code)
- Data quality gates need to validate incoming field values without pushing validation into application code
- A rule must handle differently formatted identifiers (e.g. invoice numbers with a required prefix)
- You are enforcing naming conventions on user-supplied strings before routing or enrichment
- Compliance requires that certain fields match a defined format as a condition of processing

## Composes with

- [comparison](./comparison.md) — pattern match corresponds to the `matches` operator in comparison; they are interchangeable when a name is not needed
- [predicate](./predicate.md) — a pattern match can be named as a predicate to give the format rule a reusable identity
- [logical-composition](./logical-composition.md) — pattern match results combine with other conditions when multiple format or value checks must hold together
- [fact](../fact/fact.md) — facts supply the string field value being tested

## Example

```yaml
primitive: pattern_match
field: customer.email
pattern: ".+@.+\\..+"
```

```yaml
primitive: pattern_match
field: invoice.reference_number
pattern: "^INV-[0-9]{6}$"
```
