# Event

Something that occurred in the business domain. Events are facts about the past — immutable records that trigger rules and are captured in audit logs. Unlike commands, events cannot be rejected; they have already happened and are simply observed by the system.

## Shape

```yaml
primitive: event
name: PaymentFailed
subject: payment
occurred_at: payment.failed_at
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Pascal-case name of the event, phrased in past tense |
| `subject` | entity reference | yes | The entity that the event concerns |
| `occurred_at` | field path or timestamp | yes | The authoritative timestamp when the event took place |
| `source` | string | no | The system or service that produced the event |
| `payload` | map | no | Additional fields carried by the event |

## Use when

- A business fact has been recorded and downstream rules or systems need to react to it
- You need an immutable audit trail entry for a significant domain occurrence (payment failed, invoice overdue, contract signed)
- A rule outcome depends on whether a specific thing happened in the past
- You are binding a trigger to a real-world signal from an upstream system or integration

## Composes with

- [trigger](./trigger.md) — an event is the most common trigger source; the trigger binds the event to a rule
- [emitted-event](./emitted-event.md) — rules that react to one event often emit another event as their effect
- [idempotency-key](./idempotency-key.md) — prevents duplicate event delivery from causing duplicate effects
- [entity](../entity/entity.md) — the subject of an event is always a declared entity

## Example

```yaml
primitive: event
name: PaymentFailed
subject: payment
occurred_at: payment.failed_at
source: billing-service
payload:
  payment_id: payment.id
  amount: payment.amount
  failure_reason: payment.failure_reason
```

```yaml
primitive: event
name: ContractExpired
subject: contract
occurred_at: contract.end_date
source: contract-service
payload:
  contract_id: contract.id
  customer_id: contract.customer_id
```
