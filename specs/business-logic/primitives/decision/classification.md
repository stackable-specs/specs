# Classification

A rule that assigns one value from an ordered list of classes to a target field, evaluating each class's condition in sequence and applying the first match. Classification encodes tiering, bucketing, and categorization logic in a single, scannable structure rather than nested if-else chains.

## Shape

```yaml
primitive: classification
target: customer.tier
classes:
  - value: Strategic
    when: customer.arr > 1000000
  - value: Enterprise
    when: customer.arr > 100000
  - value: SMB
    otherwise: true
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `target` | field path | yes | The field that will be written with the winning class value |
| `classes` | class list | yes | Ordered list of class entries; evaluated top-to-bottom, first match wins |
| `classes[].value` | string | yes | The value assigned to `target` when this class is selected |
| `classes[].when` | condition | no | The condition that must hold for this class to be selected; omit only on the `otherwise` entry |
| `classes[].otherwise` | boolean | no | Set to `true` on exactly one entry (typically last) to serve as the catch-all if no `when` matches |

## Use when

- A field must be assigned one of several named tiers, buckets, or categories based on data conditions
- The classification logic would otherwise require nested conditionals that are hard to read or maintain
- Thresholds or categories may change over time and should be centrally managed in one structure
- The classified value is used downstream by other rules, routing logic, or pricing engines
- You need an auditable record of which class applied and which condition triggered it

## Composes with

- [enum-decision](./enum-decision.md) — declare the valid value space with an enum-decision; classification populates it at evaluation time
- [predicate](../condition/predicate.md) — each `when` entry is a predicate evaluated against entity or fact fields
- [fact](../fact/fact.md) — facts supply the numeric or categorical values that `when` conditions compare against
- [reasoned-decision](./reasoned-decision.md) — wrap the classification result with reason codes when consumers need to understand why a particular class was assigned

## Example

```yaml
primitive: classification
target: customer.tier
classes:
  - value: Strategic
    when: customer.arr > 1000000
  - value: Enterprise
    when: customer.arr > 100000
  - value: SMB
    otherwise: true
```

```yaml
primitive: classification
target: loan_application.risk_band
classes:
  - value: Low
    when: applicant.credit_score >= 750
  - value: Medium
    when: applicant.credit_score >= 650
  - value: High
    when: applicant.credit_score >= 550
  - value: Decline
    otherwise: true
```
