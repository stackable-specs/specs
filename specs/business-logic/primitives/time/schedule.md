# Schedule

A recurring time pattern that defines when a repeating process should fire. Schedules make periodic business logic — billing runs, statement generation, renewal notices — declarative rather than buried in cron expressions or application code.

## Shape

```yaml
primitive: schedule
frequency: monthly
day: 1
time: "00:00"
timezone: account.timezone
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `frequency` | string | yes | How often the schedule fires — see valid values below |
| `day` | number or string | no | The day within the period (e.g., `1` for the 1st of the month, `Monday` for weekly) |
| `time` | string (`HH:MM`) | no | The wall-clock time at which the schedule fires |
| `timezone` | field path or IANA timezone string | no | The timezone used to interpret `day` and `time` |

Valid frequencies: `daily`, `weekly`, `monthly`, `quarterly`, `annually`, `custom`

## Use when

- A business process must repeat at a regular, predictable cadence (billing, renewals, reports)
- The firing time must respect an entity's own timezone rather than a system-wide default
- A recurring process needs to be paused, re-anchored, or overridden without changing application code
- Multiple schedules share the same frequency but differ in day, time, or timezone
- A schedule's next-fire time must be computable and storable for querying and monitoring

## Composes with

- [instant](./instant.md) — each schedule firing produces an instant that represents when the run occurred
- [deadline](./deadline.md) — a schedule may drive the evaluation of upcoming deadlines or trigger escalations
- [window](./window.md) — a schedule defines the boundary between consecutive windows (e.g., billing periods)
- [calendar-adjustment](./calendar-adjustment.md) — when a scheduled date falls on a non-business day, a calendar adjustment determines the actual run date
- [effective-period](./effective-period.md) — a schedule only fires while its associated policy or contract is within its effective period
- [event](../event/event.md) — each schedule firing emits an event (e.g., BillingCycleOpened, StatementReady)

## Example

```yaml
primitive: schedule
frequency: monthly
day: 1
time: "00:00"
timezone: account.timezone
```

```yaml
primitive: schedule
frequency: weekly
day: Monday
time: "08:00"
timezone: "America/New_York"
```
