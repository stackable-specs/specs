# Deadline

A named point in time by which an action must be completed, computed from a start instant and a duration. Deadlines make SLAs and response obligations explicit — they produce a target timestamp that can be stored, monitored, and breached.

## Shape

```yaml
primitive: deadline
start: ticket.created_at
duration: 4 business_hours
calendar: customer.support_calendar
target: ticket.first_response_due_at
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `start` | instant or field path | yes | The instant from which the duration is measured |
| `duration` | duration expression | yes | How long the subject has to act — value and unit |
| `calendar` | field path or calendar name | no | The business calendar used to resolve `business_days` or `business_hours` |
| `target` | field path | yes | The field on the entity where the computed deadline timestamp is stored |

## Use when

- A service level agreement specifies a maximum time to respond, resolve, or deliver
- A regulatory requirement mandates action within a fixed period after an initiating event
- A computed due date must be persisted on the entity so it can be queried and monitored
- Alerts or escalations need to fire a fixed interval before the deadline expires
- The deadline must respect business hours or a customer-specific holiday calendar

## Composes with

- [instant](./instant.md) — `start` is a named instant on the entity; `target` produces a new instant
- [duration](./duration.md) — the duration field borrows the duration primitive's value and unit semantics
- [calendar-adjustment](./calendar-adjustment.md) — when the computed target falls on a non-business day, a calendar adjustment shifts it
- [window](./window.md) — the period between `start` and `target` is a window within which the action must occur
- [schedule](./schedule.md) — schedules can poll for approaching or breached deadlines
- [event](../event/event.md) — a DeadlineBreached event is emitted when now exceeds the target without the required action

## Example

```yaml
primitive: deadline
start: ticket.created_at
duration: 4 business_hours
calendar: customer.support_calendar
target: ticket.first_response_due_at
```

```yaml
primitive: deadline
start: invoice.issued_at
duration: 30 days
target: invoice.due_date
```
