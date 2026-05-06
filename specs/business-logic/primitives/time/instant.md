# Instant

A single, named point in time attached to a business entity or event. Instants are the atomic timestamps that all other time primitives are built from — deadlines, windows, and schedules all anchor to or derive from named instants.

## Shape

```yaml
primitive: instant
name: order.submitted_at
type: datetime
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | field path | yes | The dot-path field name that stores this timestamp on the entity |
| `type` | string | yes | The temporal precision of the value — `datetime`, `date`, or `time` |

## Use when

- You need to record exactly when a business event occurred or a state change took place
- A deadline, window, or duration must anchor to a specific moment on a specific entity
- Audit and compliance requirements demand an immutable record of when something happened
- Two instants need to be compared (e.g., to determine if a deadline was met)
- A scheduled or time-boxed process needs a fixed start reference from an entity field

## Composes with

- [duration](./duration.md) — a duration is added to or subtracted from an instant to produce a derived instant
- [window](./window.md) — a window is defined by two instants (start and end)
- [deadline](./deadline.md) — deadlines are computed by adding a duration to a start instant
- [effective-period](./effective-period.md) — effective periods are bounded by two named instants
- [event](../event/event.md) — events carry instants that mark when they occurred
- [fact](../fact/fact.md) — instants are facts sourced from an entity or system record

## Example

```yaml
primitive: instant
name: order.submitted_at
type: datetime
```

```yaml
primitive: instant
name: invoice.due_date
type: date
```
