# Temporal Predicate

A boolean expression that evaluates time-based relationships between timestamps, durations, and the current moment. Temporal predicates make time-sensitivity explicit in business rules — encoding deadlines, windows, and age constraints as governed, auditable logic rather than computed ad-hoc.

## Shape

```yaml
primitive: temporal_predicate
expression: now <= order.delivered_at + 30d
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `expression` | string | yes | A boolean time expression using field references, `now`, duration literals (e.g. `30d`, `6h`, `1y`), and comparison operators |

**Supported functions:** `now`, `days_since(field)`, `hours_since(field)`, `date_diff(a, b, unit)`
**Duration units:** `d` (days), `h` (hours), `m` (minutes), `w` (weeks), `mo` (months), `y` (years)

## Use when

- A rule has a time window during which it is valid (e.g. refund requests within 30 days of delivery)
- An action must be triggered or blocked based on how much time has elapsed since an event
- Eligibility depends on whether a deadline has passed (e.g. trial has not yet expired)
- You are modeling SLA conditions or response-time requirements as rule predicates
- A rule must react to scheduled dates or contract renewal windows

## Composes with

- [predicate](./predicate.md) — a temporal predicate can be named as a predicate for reuse across rules
- [logical-composition](./logical-composition.md) — temporal expressions combine with non-temporal conditions in compound eligibility checks
- [comparison](./comparison.md) — temporal predicates extend comparison semantics to time types and duration arithmetic
- [change-predicate](./change-predicate.md) — the timestamp of a state change is a common `now`-relative anchor for temporal predicates
- [fact](../fact/fact.md) — facts supply the timestamp fields used in temporal expressions

## Example

```yaml
primitive: temporal_predicate
expression: now <= order.delivered_at + 30d
```

```yaml
primitive: temporal_predicate
expression: days_since(subscription.trial_started_at) >= 14
```
