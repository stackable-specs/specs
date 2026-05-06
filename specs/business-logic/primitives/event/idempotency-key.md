# Idempotency Key

Ensures an effect is applied exactly once even if the triggering event is delivered multiple times. In distributed systems, events may be replayed due to retries, network failures, or at-least-once delivery guarantees — an idempotency key prevents duplicate processing from producing duplicate outcomes.

## Shape

```yaml
primitive: idempotency_key
value: invoice.id + ":collection_task"
on_duplicate: skip
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `value` | expression | yes | A string expression that uniquely identifies a single execution of the effect; typically composed from entity IDs and an operation name |
| `on_duplicate` | enum | yes | The behavior when a key collision is detected. One of: `skip`, `update`, `error`, `merge` |
| `ttl` | duration | no | How long the key is retained before expiring; after expiry, the same key is treated as new |
| `scope` | string | no | Namespace for the key registry; defaults to rule name if omitted |

## on_duplicate options

| Value | Behavior |
|-------|----------|
| `skip` | Silently discard the duplicate execution; no effect is applied |
| `update` | Re-apply the effect with the latest payload, overwriting the prior result |
| `error` | Raise an explicit error so the duplicate can be investigated |
| `merge` | Combine the new payload with the existing record rather than replacing it |

## Use when

- A rule creates external side effects (tasks, charges, notifications) that must not be duplicated
- The event source uses at-least-once delivery and replay is expected during failure recovery
- A workflow step is triggered by a schedule and could fire twice due to clock drift or clock skew
- You need a traceable record that a specific action was already taken for a given entity instance

## Composes with

- [event](./event.md) — idempotency keys guard against duplicate delivery of source events
- [emitted-event](./emitted-event.md) — prevents the same rule execution from emitting the same event multiple times
- [retry](../exception/retry.md) — retry policies and idempotency keys work together; retries re-use the same key so only one successful execution counts
- [trigger](./trigger.md) — the trigger context provides the fields that compose the key value

## Example

```yaml
primitive: idempotency_key
value: invoice.id + ":collection_task"
on_duplicate: skip
ttl: 90 days
scope: collections-rule
```

```yaml
primitive: idempotency_key
value: payment.id + ":failure_notification:" + payment.attempt_count
on_duplicate: error
ttl: 7 days
```
