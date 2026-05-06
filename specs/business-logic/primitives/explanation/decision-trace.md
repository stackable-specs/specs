# Decision Trace

A full record of which rules fired, which conditions matched, and what outcome was selected — for debugging and audit. The decision trace is the complete execution log of a single rule evaluation, structured so that engineers can diagnose unexpected outcomes, compliance teams can verify correct behavior, and automated monitors can detect regressions.

## Shape

```yaml
primitive: decision_trace
captures:
  - evaluated_rules
  - matched_conditions
  - skipped_conditions
  - selected_outcome
  - overrides_applied
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `captures` | string list | yes | Named dimensions of the evaluation to record. See capture options below |
| `trace_id` | string | no | Unique identifier for this trace instance, correlatable with the originating request |
| `rule_id` | string | no | The rule whose evaluation this trace records |
| `rule_version` | integer | no | The version of the rule evaluated |
| `evaluated_at` | datetime | no | UTC timestamp of evaluation |
| `triggered_by` | string | no | The actor or system event that initiated the evaluation |

## Capture options

| Value | Description |
|-------|-------------|
| `evaluated_rules` | All rules that were considered, whether or not they fired |
| `matched_conditions` | Predicates and conditions that evaluated to true |
| `skipped_conditions` | Conditions that were not evaluated due to short-circuit logic |
| `selected_outcome` | The final decision or effect that was chosen |
| `overrides_applied` | Any manual override that changed the automated outcome |
| `reason_codes` | The reason codes produced by matched rules |
| `evidence` | The fact values that influenced matched conditions |
| `policy_references` | Policy documents cited by the evaluated rules |

## Use when

- A decision outcome is unexpected and you need a step-by-step trace of which rules fired and why
- A rule evaluation must be replayed exactly — with the same inputs and logic — to verify correctness after a rule change
- Compliance requires a per-decision record demonstrating that the correct rule version was evaluated with the correct inputs
- You are building automated regression tests that compare decision traces before and after a rule change
- An override was applied and you need to record both the automated outcome and the override alongside the reason

## Composes with

- [reason-code](./reason-code.md) — reason codes are one of the most important capture dimensions; the decision trace makes them queryable by rule, condition, and outcome
- [evidence](./evidence.md) — evidence items are the per-fact records within a decision trace, preserving the exact input values the rule saw
- [explanation-template](./explanation-template.md) — a decision trace is the data source from which an explanation template is rendered for any audience
- [../governance/audit-requirement](../governance/audit-requirement.md) — an audit requirement causes the decision trace to be persisted to a durable store; the trace is the artifact the requirement governs

## Example

```yaml
primitive: decision_trace
trace_id: trc_8af3c2
rule_id: refund_eligibility
rule_version: 3
evaluated_at: "2026-02-15T10:23:44Z"
triggered_by: customer_support_agent
captures:
  - evaluated_rules
  - matched_conditions
  - skipped_conditions
  - selected_outcome
  - reason_codes
  - evidence
  - overrides_applied
```

```yaml
primitive: decision_trace
trace_id: trc_9bc7d1
rule_id: pricing_v2
rule_version: 2
evaluated_at: "2026-03-10T14:05:22Z"
triggered_by: checkout_service
captures:
  - evaluated_rules
  - matched_conditions
  - selected_outcome
  - reason_codes
  - policy_references
```
