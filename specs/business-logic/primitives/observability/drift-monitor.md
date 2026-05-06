# Drift Monitor

Detects when a metric shifts significantly from its historical baseline, signaling that rule behavior may have changed unexpectedly. Drift monitors are the automated canary for rule health — they catch silent degradations that threshold alerts miss, such as a gradual decline in approval rate that never crosses a hard threshold but indicates something has changed in the data, population, or rule.

## Shape

```yaml
primitive: drift_monitor
metric: approval_rate
baseline_period: previous_30_days
alert_when:
  - current_value changes_by > 20%
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `metric` | string | yes | The `name` of the metric primitive to monitor for drift |
| `baseline_period` | enum or duration | yes | The historical window used to compute the baseline. Common values: `previous_7_days`, `previous_30_days`, `same_period_last_year`, or an ISO 8601 duration like `P30D` |
| `alert_when` | predicate list | yes | Conditions comparing the current metric value to the baseline that trigger a drift alert. Use `changes_by >` for relative change, `changes_by_absolute >` for absolute delta |
| `direction` | enum | no | Which direction of change triggers the alert. One of: `increase`, `decrease`, `either`. Defaults to `either` |
| `evaluation_frequency` | duration | no | How often the current value is compared to the baseline (e.g., `1 hour`, `6 hours`, `1 day`). Defaults to `1 day` |
| `minimum_sample_size` | integer | no | Minimum number of executions required in the current window before drift is evaluated; prevents false positives on low-traffic rules |
| `notify` | string list | no | Roles or channels to notify when drift is detected. If omitted, delegates to the referenced `alert` primitive |
| `owner` | string | no | Role responsible for investigating and resolving detected drift |

## Use when

- A rule change has been deployed and you want automated detection of behavioral shifts beyond what a fixed threshold alert provides
- A rule operates in a context where the input population changes over time (seasonal patterns, product changes, new customer segments) and you need to distinguish normal variation from unexpected drift
- You are monitoring a rule that has no obvious hard threshold to alert on, but any significant deviation from its historical behavior is worth investigating
- A rollout is in progress and you want to detect if the new rule version behaves materially differently from the baseline version on the same cohort
- You want to set up long-running monitors that adapt their baseline as the rule matures, rather than requiring manual threshold updates

## Composes with

- [metric](./metric.md) — the drift monitor references a named metric as the signal it watches; the metric definition determines the data source, formula, and window
- [alert](./alert.md) — a drift monitor fires an alert when it detects significant drift; compose them so the alert handles notification routing, cooldown, and runbook linkage
- [../governance/rollout-control](../governance/rollout-control.md) — drift monitors are the primary automated safety check during a phased rollout; scope the monitored metric to the rollout cohort to isolate the signal
- [../governance/kill-switch](../governance/kill-switch.md) — when drift is detected and confirms a rule problem, the kill switch is the remediation tool; drift monitor runbooks should reference the relevant kill switch

## Example

```yaml
primitive: drift_monitor
metric: approval_rate
baseline_period: previous_30_days
alert_when:
  - current_value changes_by > 20%
direction: either
evaluation_frequency: 6 hours
minimum_sample_size: 100
notify:
  - policy_owner
  - engineering_oncall_channel
owner: FinanceOps
```

```yaml
primitive: drift_monitor
metric: refund_override_rate
baseline_period: previous_7_days
alert_when:
  - current_value changes_by > 15%
direction: increase
evaluation_frequency: 1 hour
minimum_sample_size: 50
notify:
  - finance_ops_channel
owner: FinanceManager
```
