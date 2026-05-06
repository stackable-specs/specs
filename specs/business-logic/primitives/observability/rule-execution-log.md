# Rule Execution Log

A per-execution record of rule id, version, timing, and result. The rule execution log is the raw, high-frequency event stream produced every time a rule is evaluated — the foundational data source from which metrics are aggregated, alerts are derived, and debugging queries are run.

## Shape

```yaml
primitive: rule_execution_log
capture:
  - rule_id
  - version
  - execution_time_ms
  - result
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `capture` | string list | yes | Named fields to record for each rule execution. See capture options below |
| `destination` | string | no | Log sink that receives execution records (e.g., `openobserve`, `s3://execution-logs`, `stdout`) |
| `sampling_rate` | float | no | Fraction of executions to record (0.0–1.0). Defaults to `1.0` (all executions). Useful for very high-throughput rules |
| `include_slow_executions` | boolean | no | If true, executions exceeding `slow_threshold_ms` are always recorded regardless of `sampling_rate` |
| `slow_threshold_ms` | integer | no | Millisecond threshold above which an execution is considered slow and flagged in the log |

## Capture options

| Value | Description |
|-------|-------------|
| `rule_id` | The stable identifier of the rule that was evaluated |
| `version` | The rule version that was active at evaluation time |
| `execution_time_ms` | Wall-clock time in milliseconds from rule entry to result |
| `result` | The outcome produced: approved, denied, flagged, etc. |
| `input_hash` | A hash of the input facts — for deduplication and replay |
| `actor_id` | The user or service that triggered evaluation |
| `timestamp` | UTC instant of evaluation |
| `error` | Any exception or evaluation error that occurred |

## Use when

- You need a raw, queryable record of every rule evaluation to support ad-hoc debugging and performance analysis
- You are building metrics (approval rates, error rates, p99 latency) and need a reliable event source to aggregate from
- A rule is behaving unexpectedly in production and you need per-request execution data to identify when the problem started
- You want to compare execution timing across rule versions to detect performance regressions introduced by a rule change
- Sampling is needed on very high-throughput rules where full logging is cost-prohibitive

## Composes with

- [metric](./metric.md) — metrics are computed by aggregating rule execution log events over time; the log is the canonical upstream source
- [alert](./alert.md) — alert thresholds reference metrics derived from the execution log; the log is the data that ultimately triggers alerts
- [../explanation/decision-trace](../explanation/decision-trace.md) — the execution log is a lightweight, high-volume record; the decision trace is the full, structured record for a specific execution — they are complementary, not redundant
- [../governance/audit-requirement](../governance/audit-requirement.md) — audit requirements and execution logs overlap in what they capture; the execution log is for operational observability, the audit record is for compliance — both may exist for the same rule

## Example

```yaml
primitive: rule_execution_log
capture:
  - rule_id
  - version
  - execution_time_ms
  - result
  - actor_id
  - timestamp
  - error
destination: openobserve
sampling_rate: 1.0
include_slow_executions: true
slow_threshold_ms: 200
```

```yaml
primitive: rule_execution_log
capture:
  - rule_id
  - version
  - execution_time_ms
  - result
  - input_hash
destination: s3://prod-execution-logs
sampling_rate: 0.1
include_slow_executions: true
slow_threshold_ms: 500
```
