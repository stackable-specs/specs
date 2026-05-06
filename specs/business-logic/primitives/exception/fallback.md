# Fallback

The outcome produced when no other rule in a rule set matches. Every rule set should declare a fallback to prevent undefined behavior. A fallback makes the default explicit, traceable, and deliberate — rather than leaving the system in an unspecified state when no condition is satisfied.

## Shape

```yaml
primitive: fallback
when: no_rule_matched
outcome:
  customer.tier: SMB
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `when` | enum or condition | yes | The condition that activates the fallback. Use `no_rule_matched` for a catch-all, or a condition expression for a more specific guard |
| `outcome` | outcome map | yes | The field assignments or effects to apply when the fallback activates |
| `emit` | emitted event reference | no | An optional event to emit when the fallback fires, useful for alerting or monitoring |
| `log_reason` | boolean | no | If true, records a log entry noting that no rule matched and the fallback was used |

## Use when

- A rule set evaluates conditions across a range of cases and a gap in coverage must produce a safe, defined result
- The business has an established "default" treatment that applies unless a more specific rule takes precedence
- You want to detect uncovered cases through monitoring without crashing the workflow
- A classification or routing rule must always assign the entity to some category even when no criteria match

## Composes with

- [exception](./exception.md) — exceptions are evaluated before the fallback; if no exception applies and no rule matches, the fallback fires
- [emitted-event](../event/emitted-event.md) — emit a `NoRuleMatched` or `FallbackApplied` event so downstream monitoring can track coverage gaps
- [decision](../decision/decision.md) — a decision primitive typically owns the rule set and declares the fallback as its default branch
- [explanation](../explanation/rationale.md) — the fallback outcome should always be explainable; pair with a rationale describing why the default was chosen

## Example

```yaml
primitive: fallback
when: no_rule_matched
outcome:
  customer.tier: SMB
log_reason: true
emit:
  name: CustomerTierDefaulted
  payload:
    customer_id: customer.id
    defaulted_tier: SMB
```

```yaml
primitive: fallback
when: no_rule_matched
outcome:
  invoice.collection_strategy: StandardDunning
  invoice.escalation_days: 30
log_reason: true
```
