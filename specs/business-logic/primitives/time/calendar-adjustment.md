# Calendar Adjustment

A rule that shifts a computed date when it falls on a non-business day. Calendar adjustments separate the raw date arithmetic from the business convention that governs what to do when a deadline or schedule lands on a weekend or public holiday.

## Shape

```yaml
primitive: calendar_adjustment
input: invoice.due_date
calendar: banking_calendar
if_non_business_day: next_business_day
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `input` | field path or expression | yes | The date or datetime to be adjusted |
| `calendar` | calendar name or field path | yes | The business calendar that defines which days are non-business days |
| `if_non_business_day` | string | yes | The convention to apply when the input date is a non-business day — see valid values below |

Valid options for `if_non_business_day`: `next_business_day`, `previous_business_day`, `no_adjustment`

## Use when

- A deadline or payment date computed by adding a duration might fall on a weekend or public holiday
- Regulatory requirements specify a particular rolling convention (e.g., modified following for bond settlements)
- Different entity types must respect different calendars (banking vs. customer support vs. stock exchange)
- A schedule fires on a fixed calendar day but must be moved when that day is not a business day
- The adjusted date must be stored separately from the raw computed date for audit purposes

## Composes with

- [deadline](./deadline.md) — deadlines are the most common consumer of calendar adjustments after computing a target date
- [schedule](./schedule.md) — schedules use calendar adjustments to determine the actual run date when a scheduled day is a holiday
- [duration](./duration.md) — when a duration is expressed in `business_days`, the calendar drives which days are counted
- [instant](./instant.md) — the output of a calendar adjustment is a new instant stored on the entity
- [effective-period](./effective-period.md) — effective period boundaries may be adjusted so they fall on business days

## Example

```yaml
primitive: calendar_adjustment
input: invoice.due_date
calendar: banking_calendar
if_non_business_day: next_business_day
```

```yaml
primitive: calendar_adjustment
input: subscription.renewal_date
calendar: account.country_calendar
if_non_business_day: previous_business_day
```
