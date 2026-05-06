# Trigger

The signal that causes a rule to fire. A trigger is the binding between a source — event, command, schedule, state change, or other signal — and a rule. Without a declared trigger, a rule has no entry point and cannot execute.

## Shape

```yaml
primitive: trigger
type: event
name: PaymentFailed
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | enum | yes | The category of signal that activates the rule. One of: `event`, `command`, `schedule`, `state_change`, `query`, `manual`, `external_signal` |
| `name` | string | yes | The specific event name, command name, schedule expression, or signal identifier |
| `filter` | condition expression | no | An optional guard that must evaluate true before the rule fires; reduces noise when many instances match the signal |
| `debounce` | duration | no | Suppresses repeat firings within the given window; useful for high-frequency state changes |

## Trigger types

| Type | Description |
|------|-------------|
| `event` | Fires when a named domain event is received |
| `command` | Fires when a named command is issued by an actor |
| `schedule` | Fires on a cron or interval expression (e.g. `every day at 09:00`) |
| `state_change` | Fires when a field transitions from one value to another |
| `query` | Fires when a query result crosses a threshold (e.g. overdue invoice count > 0) |
| `manual` | Fires only when a human explicitly invokes the rule |
| `external_signal` | Fires on a webhook or integration event from an outside system |

## Use when

- You need to declare what causes a rule to activate
- A rule should fire on a recurring schedule rather than in response to a single event
- You want to filter a high-volume event stream so the rule only fires for relevant instances
- A state field crossing a threshold (e.g. `days_overdue` going from 29 to 30) should initiate a workflow

## Composes with

- [event](./event.md) — the most common trigger source; wraps a named domain event
- [command](./command.md) — triggers a rule when an actor issues a specific command
- [idempotency-key](./idempotency-key.md) — prevents the same trigger firing from causing duplicate rule executions
- [exception](../exception/exception.md) — exceptions can suppress a trigger from reaching the rule body under certain conditions

## Example

```yaml
primitive: trigger
type: event
name: PaymentFailed
filter: payment.attempt_count >= 3
```

```yaml
primitive: trigger
type: schedule
name: "every day at 08:00"
filter: invoice.status = Overdue
```

```yaml
primitive: trigger
type: state_change
name: account.credit_hold
filter: account.credit_hold changed_to true
```
