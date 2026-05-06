# Fact

A single named input value consumed by a rule. Facts are the raw material of business logic — predicates test them, formulas compute from them, and audit records capture them. Declaring facts explicitly makes a rule's data dependencies visible and testable.

## Shape

```yaml
primitive: fact
name: <field_path>
type: <type>
source: <service_or_system>
required: true | false
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | field path | yes | Dot-notation path to the value (e.g. `order.total_amount`) |
| `type` | string | yes | One of: `string`, `number`, `boolean`, `date`, `datetime`, `money`, `enum`, `object`, `collection` |
| `source` | string | no | The system or service that owns this value |
| `required` | boolean | yes | If true, the rule cannot proceed without this value — pair with [missing-fact](./missing-fact.md) to define the fallback |

## Use when

- You need to declare what data a rule depends on so it can be tested, documented, and traced
- A predicate, formula, or decision needs a named input
- You want to make data provenance explicit (where does this value come from?)

## Composes with

- [derived-fact](./derived-fact.md) — when the value must be computed rather than sourced directly
- [missing-fact](./missing-fact.md) — defines what happens if this fact is absent
- [fact-source](./fact-source.md) — declares trustworthiness and freshness of the source
- [predicate](../condition/predicate.md) — tests the value of this fact
- [formula](../calculation/formula.md) — uses this fact as an operand

## Example

```yaml
primitive: fact
name: order.total_amount
type: money
source: order_service
required: true
```

```yaml
primitive: fact
name: customer.tier
type: enum
source: crm_service
required: false
```
