# Alert

Fires when a metric crosses a threshold, notifying a defined owner. Alerts translate passive metric monitoring into active signals — ensuring that when a rule begins misbehaving (too many overrides, too many errors, too much latency), the right person or team is notified before the problem compounds.

## Shape

```yaml
primitive: alert
trigger_when:
  - refund_override_rate > 0.10
notify:
  - policy_owner
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `trigger_when` | predicate list | yes | One or more metric conditions that, when true, fire the alert. Predicates reference metric names defined by `metric` primitives |
| `notify` | string list | yes | Roles, channels, or identities to notify when the alert fires |
| `severity` | enum | no | Urgency classification. One of: `critical`, `high`, `medium`, `low`. Affects routing and escalation behavior |
| `cooldown` | duration | no | Minimum time between repeated firings of the same alert (e.g., `1 hour`, `30 minutes`) to prevent notification storms |
| `resolve_when` | predicate list | no | Conditions under which a fired alert automatically resolves. If omitted, the alert must be manually acknowledged |
| `runbook_url` | string | no | Link to the runbook describing how to investigate and remediate this alert |
| `source_metric` | string | no | The `name` of the metric primitive being evaluated. Makes the dependency explicit for tooling |

## Use when

- A metric crossing a threshold indicates a rule is behaving outside its expected operational envelope
- You want to be notified when a rollout causes unexpected shifts in approval rate, error rate, or latency before it affects the full user population
- A kill switch or rollout rollback should be human-initiated but human-triggered by data — alerts are the data trigger
- Compliance or finance teams need proactive notification when override rates or error rates exceed acceptable bounds
- You are setting up a monitoring layer for a new rule family and need to define the health signals before the rule goes live

## Composes with

- [metric](./metric.md) — alerts reference named metrics as their trigger signal; a metric must be defined before an alert can reference it
- [drift-monitor](./drift-monitor.md) — drift monitors produce alerts when a metric deviates from its baseline; the alert primitive defines the notification behavior for that event
- [../governance/kill-switch](../governance/kill-switch.md) — alerts are the operational trigger that tells an on-call engineer to activate a kill switch; wire alert runbooks to kill switch procedures
- [../governance/rollout-control](../governance/rollout-control.md) — an alert scoped to a rollout cohort can detect problems in the new rule version before traffic is shifted to the full population

## Example

```yaml
primitive: alert
trigger_when:
  - refund_override_rate > 0.10
notify:
  - policy_owner
  - finance_ops_channel
severity: high
cooldown: 1 hour
resolve_when:
  - refund_override_rate <= 0.08
runbook_url: https://wiki.internal/runbooks/refund-override-rate-spike
source_metric: refund_override_rate
```

```yaml
primitive: alert
trigger_when:
  - pricing_rule_p99_latency > 400
  - pricing_rule_error_rate > 0.01
notify:
  - engineering_oncall_channel
severity: critical
cooldown: 15 minutes
runbook_url: https://wiki.internal/runbooks/pricing-rule-degradation
```
