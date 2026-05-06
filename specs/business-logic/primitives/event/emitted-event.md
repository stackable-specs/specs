# Emitted Event

An event produced as an effect of a rule executing. Emitted events propagate outcomes to downstream systems and rules, forming the connective tissue of event-driven business logic. Unlike source events, emitted events originate inside the rule engine rather than from an external system.

## Shape

```yaml
primitive: emit_event
name: InvoiceSubmitted
payload:
  invoice_id: invoice.id
  submitted_at: now
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Pascal-case name of the event to emit, phrased in past tense |
| `payload` | map | yes | Key-value pairs attached to the emitted event; values may reference entity fields or system variables like `now` |
| `destination` | string | no | The topic, queue, or system that should receive the event; defaults to the platform event bus |
| `delay` | duration | no | Optional delay before the event is published (e.g. `24 hours` for a deferred notification) |
| `condition` | condition expression | no | If present, the event is only emitted when this condition holds at rule execution time |

## Use when

- A rule produces an outcome that other rules or downstream services need to observe
- You want to decouple the producing rule from its consumers without direct coupling
- A workflow step completion should be visible in audit logs as a first-class domain event
- An outcome should trigger a chain of reactions across multiple bounded contexts (billing, notifications, reporting)

## Composes with

- [event](./event.md) — defines the schema and semantics of the event being emitted; the emitted event becomes a source event for downstream rules
- [trigger](./trigger.md) — the emitted event can serve as a trigger for subsequent rules
- [idempotency-key](./idempotency-key.md) — prevents duplicate rule executions from emitting the same event twice
- [compensation](../exception/compensation.md) — a compensation action may emit a reversal event when a workflow step fails

## Example

```yaml
primitive: emit_event
name: InvoiceSubmitted
payload:
  invoice_id: invoice.id
  customer_id: invoice.customer_id
  submitted_at: now
  total_amount: invoice.total
destination: billing.events
```

```yaml
primitive: emit_event
name: AccountPlacedOnCreditHold
payload:
  account_id: account.id
  triggered_by: overdue_invoice.id
  hold_placed_at: now
condition: account.overdue_balance > account.credit_limit
destination: collections.events
```
