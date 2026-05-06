# Metric

An aggregate measure computed from rule outcomes over time. Metrics translate the raw event stream from rule execution logs into business-meaningful signals — approval rates, override frequencies, error rates, latency percentiles — that indicate whether a rule is healthy, effective, and behaving as intended.

## Shape

```yaml
primitive: metric
name: refund_override_rate
formula: override_count / refund_decision_count
window: rolling_30_days
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Snake-case identifier for the metric, stable across versions |
| `formula` | expression | yes | The computation that produces the metric value from aggregated execution log data |
| `window` | enum or duration | yes | Time window over which the formula is evaluated. Common values: `rolling_7_days`, `rolling_30_days`, `calendar_month`, `rolling_24_hours`, or an ISO 8601 duration like `P7D` |
| `source_rule` | string | no | The `rule_id` whose execution log is the data source for this metric. Omit for cross-rule metrics |
| `unit` | string | no | Unit of the metric value (e.g., `ratio`, `count`, `milliseconds`, `percent`) |
| `description` | string | no | Human-readable explanation of what the metric measures and why it matters |
| `owner` | string | no | Role or team responsible for acting on this metric when it moves |

## Use when

- You need to monitor whether a rule's behavior is stable over time, not just correct on individual evaluations
- Business stakeholders need a health signal for a decision process (e.g., "what percentage of refunds are being overridden?")
- You want to compare rule behavior before and after a version change by tracking a metric across both versions
- A drift monitor or alert needs a defined metric as its input signal
- You are building a dashboard for a rule family and need to define which aggregate measures appear on it

## Composes with

- [rule-execution-log](./rule-execution-log.md) — rule execution logs are the raw event source that metric formulas aggregate over; the metric is meaningless without a reliable log upstream
- [alert](./alert.md) — alerts reference metric names as their trigger condition; a metric must be defined before an alert can reference it
- [drift-monitor](./drift-monitor.md) — drift monitors track a named metric against its historical baseline; the metric definition determines what is being compared
- [../governance/rollout-control](../governance/rollout-control.md) — during a phased rollout, metrics scoped per rule version reveal whether the new version is performing better or worse than the fallback

## Example

```yaml
primitive: metric
name: refund_override_rate
formula: override_count / refund_decision_count
window: rolling_30_days
source_rule: refund_eligibility
unit: ratio
description: "Fraction of automated refund decisions that were manually overridden by an agent or manager."
owner: FinanceOps
```

```yaml
primitive: metric
name: pricing_rule_p99_latency
formula: percentile(execution_time_ms, 99)
window: rolling_24_hours
source_rule: pricing_v2
unit: milliseconds
description: "99th percentile execution latency for the pricing rule, used to detect performance regressions."
owner: EngineeringPlatform
```
