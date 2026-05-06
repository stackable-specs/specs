# Set Field

Assigns a scalar value to a single field on an entity. This is the most atomic write effect — it changes exactly one field and nothing else. Use it when a rule outcome needs to stamp a status, flag, or computed value onto the subject entity without creating or deleting records.

## Shape

```yaml
primitive: set_field
field: invoice.status
value: Submitted
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `field` | field path | yes | Dot-path to the field being written, in the form `entity.field` |
| `value` | scalar or field path | yes | The value to assign; may be a literal string, number, boolean, or a reference to another field |
| `condition` | predicate | no | Guard expression; the write is skipped if the condition evaluates to false |

## Use when

- A rule outcome changes the status, stage, or category of an entity (e.g., mark an invoice as `Submitted`, an order as `Cancelled`)
- A computed or derived value needs to be persisted back to the entity after calculation
- A flag must be toggled in response to a business event (e.g., set `account.credit_hold = true`)
- You want the narrowest possible write scope — changing one field, nothing else
- Downstream systems poll the field value rather than subscribing to events

## Composes with

- [transition](../state/transition.md) — state transitions commonly emit a `set_field` effect to persist the new state value
- [update-record](./update-record.md) — when multiple fields must change together, group them under `update_record` instead
- [fact](../fact/fact.md) — the field being written often mirrors a derived fact that was computed by a rule
- [event](../event/event.md) — a `set_field` effect is frequently paired with an emitted event so that other systems can react

## Example

```yaml
primitive: set_field
field: invoice.status
value: Submitted
```

```yaml
primitive: set_field
field: account.credit_hold
value: true
condition: account.days_past_due > 90
```
