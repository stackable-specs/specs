# Window

A bounded interval between two instants that defines when something is eligible, observable, or actionable. Windows make time ranges a first-class concept so that rules can ask whether an event falls inside, before, or after a specific period.

## Shape

```yaml
primitive: window
start: order.delivered_at
end: order.delivered_at + 30d
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `start` | instant or expression | yes | The inclusive start of the window — a named instant or a derived expression |
| `end` | instant or expression | yes | The exclusive end of the window — a named instant or a derived expression |

## Use when

- Eligibility for an action depends on whether the current time falls within a defined range
- A return, appeal, or dispute period opens at one event and closes at another
- Rules must evaluate which records were active, submitted, or valid during a specific interval
- A billing cycle, reporting period, or subscription window must be compared against transaction timestamps
- Two time windows need to be checked for overlap (e.g., concurrent leave requests)

## Composes with

- [instant](./instant.md) — `start` and `end` reference or derive from named instants
- [duration](./duration.md) — the window span is often expressed as start + duration to produce the end
- [deadline](./deadline.md) — a deadline marks the end of a window within which action must be taken
- [effective-period](./effective-period.md) — an effective period is a window scoped to the validity of a policy or price
- [as-of-evaluation](./as-of-evaluation.md) — as-of evaluation checks whether a given as-of date falls inside a window
- [condition](../condition/condition.md) — conditions test whether now or another instant is within the window

## Example

```yaml
primitive: window
start: order.delivered_at
end: order.delivered_at + 30d
```

```yaml
primitive: window
start: subscription.trial_start_at
end: subscription.trial_end_at
```
