# Duration

A named length of time expressed as a quantity and a unit. Durations are not anchored to any specific point on a calendar — they represent a reusable time span that can be added to an instant to produce a deadline or window boundary.

## Shape

```yaml
primitive: duration
value: 30
unit: days
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `value` | number | yes | The numeric length of the time span |
| `unit` | string | yes | The unit of measurement — see valid values below |

Valid units: `minutes`, `hours`, `days`, `business_days`, `weeks`, `months`, `years`

## Use when

- A business rule specifies a length of time without yet anchoring it to a specific start point
- The same time span is reused across multiple deadlines, windows, or schedules
- The distinction between calendar days and business days is meaningful for the rule
- A payment term, return window, response SLA, or trial period must be expressed precisely
- A duration is parameterised by entity attributes (e.g., the credit term defined on the account)

## Composes with

- [instant](./instant.md) — a duration is added to an instant to compute a derived timestamp
- [deadline](./deadline.md) — deadline uses a duration to calculate how long the subject has to act
- [window](./window.md) — the span between a window's start and end can be expressed as a duration
- [schedule](./schedule.md) — schedules may use durations to define look-back or look-ahead periods
- [calendar-adjustment](./calendar-adjustment.md) — when a duration uses `business_days`, a calendar adjustment resolves which days count

## Example

```yaml
primitive: duration
value: 30
unit: days
```

```yaml
primitive: duration
value: 4
unit: business_hours
```

```yaml
primitive: duration
value: 12
unit: months
```
