# Update Record

Modifies specific fields on an existing entity instance identified by a key. Unlike `set_field`, which targets a single field path, `update_record` groups multiple field changes into one atomic write. Use it when several fields must change together to preserve consistency — changing one without the others would leave the record in an invalid intermediate state.

## Shape

```yaml
primitive: update_record
entity: Account
id: account.id
changes:
  status: Suspended
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `entity` | string | yes | Pascal-case name of the entity type being updated |
| `id` | field path | yes | Field path that resolves to the identifier of the target record |
| `changes` | map | yes | Key-value pairs of field names to overwrite; values may be literals or field path references |
| `condition` | predicate | no | Guard expression; the update is skipped if the condition evaluates to false |

## Use when

- Multiple fields on an existing record must change atomically as the consequence of a rule (e.g., suspend an account and record the suspension timestamp in one operation)
- A rule outcome modifies a related entity rather than the primary subject (e.g., update the parent `Order` when a child `OrderLine` is cancelled)
- The target record is identified dynamically at runtime via a field path reference rather than a literal ID
- You need a clear, named boundary around a set of field writes for auditability and rollback purposes
- Partial updates to a record that must not touch fields outside the declared `changes` set

## Composes with

- [set-field](./set-field.md) — use `set_field` for single-field changes; use `update_record` when three or more fields must change together
- [entity](../entity/entity.md) — the `entity` field must reference a declared entity primitive
- [transition](../state/transition.md) — state transitions frequently emit an `update_record` effect to persist state and metadata simultaneously
- [event](../event/event.md) — pair with an emitted event so downstream systems observe the change without polling the record
- [fact](../fact/fact.md) — the `id` field path commonly resolves through a relationship fact (e.g., `order.account_id`)

## Example

```yaml
primitive: update_record
entity: Account
id: account.id
changes:
  status: Suspended
  suspended_at: now
  suspension_reason: CreditLimitExceeded
```

```yaml
primitive: update_record
entity: Subscription
id: subscription.id
changes:
  status: Cancelled
  cancelled_at: now
  cancellation_reason: PaymentFailure
  auto_renew: false
```
