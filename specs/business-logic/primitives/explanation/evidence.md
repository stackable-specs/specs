# Evidence

A specific fact value that influenced a decision, recorded alongside the outcome for explainability and audit. Evidence preserves the exact input the rule saw at evaluation time — not a summary, but the raw value — so that the decision can be explained, reproduced, and defended even after the underlying data has changed.

## Shape

```yaml
primitive: evidence
fact: order.delivered_at
value: "2026-01-01"
used_by: refund_eligibility
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `fact` | field path | yes | Dot-path to the fact that was read during rule evaluation |
| `value` | any | yes | The actual value of the fact at evaluation time, serialized as a scalar or structure |
| `used_by` | string | yes | The `rule_id` of the rule that read this fact and was influenced by it |
| `relevant_condition` | string | no | The specific predicate or condition within the rule that evaluated this fact (e.g., `order.delivered_at < cutoff_date`) |
| `evaluated_at` | datetime | no | UTC timestamp at which the fact was read, for audits where data may change between request and response |

## Use when

- A rule produces a decision and the specific facts that drove that decision must be recorded for explainability
- A customer, agent, or regulator asks "why was this decision made?" and you need to show the exact values the system saw
- You want to detect data drift between evaluation time and the time an audit is run (the stored evidence value differs from the current data value)
- A dispute arises over a decision and you need to prove what the rule evaluated, not what the data looks like today
- You are building a decision trace and need to attach the relevant inputs to each fired rule

## Composes with

- [reason-code](./reason-code.md) — reason codes state what conclusion was reached; evidence records the specific values that justified it — together they form a complete, auditable explanation
- [decision-trace](./decision-trace.md) — a decision trace collects all evidence items produced during evaluation into a single structured record
- [explanation-template](./explanation-template.md) — evidence values can be interpolated into explanation templates to show users not just the reason but the specific data behind it
- [../governance/audit-requirement](../governance/audit-requirement.md) — `input_facts` in an audit requirement causes evidence items to be captured and persisted alongside the decision record

## Example

```yaml
primitive: evidence
fact: order.delivered_at
value: "2026-01-01"
used_by: refund_eligibility
relevant_condition: "order.delivered_at < refund_cutoff_date"
evaluated_at: "2026-02-15T10:23:44Z"
```

```yaml
primitive: evidence
fact: customer.loyalty_tier
value: "Gold"
used_by: pricing_discount
relevant_condition: "customer.loyalty_tier in [Gold, Platinum]"
evaluated_at: "2026-02-15T10:23:44Z"
```
