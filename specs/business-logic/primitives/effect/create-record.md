# Create Record

Creates a new entity instance as a side effect of a rule firing. The new record is fully initialized from a declared set of field values, which may be literals or references to fields on the triggering entity. Use it when a business rule must produce a new persistent object — a task, a follow-up, a child record — rather than modifying an existing one.

## Shape

```yaml
primitive: create_record
entity: Task
values:
  type: CollectionFollowUp
  owner: CollectionsQueue
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `entity` | string | yes | Pascal-case name of the entity type to instantiate |
| `values` | map | yes | Key-value pairs of field names to assign on the new record; values may be literals or field path references |
| `id_field` | string | no | Name of the field that will hold the auto-generated identifier; defaults to the entity's declared `id_field` |
| `condition` | predicate | no | Guard expression; record creation is skipped if the condition evaluates to false |

## Use when

- A rule must spawn a downstream work item (task, case, ticket, follow-up) as its consequence
- A business event requires a new child record to be created under an existing parent (e.g., create a `CreditMemo` when a return is approved)
- An audit or compliance rule requires a new record for every occurrence of a condition (e.g., create a `FraudAlert` record when a suspicious transaction is detected)
- Multiple new records of the same type may need to be created in one rule execution (e.g., one `Task` per overdue line item)
- The newly created record will itself be subject to subsequent rules

## Composes with

- [entity](../entity/entity.md) — the `entity` field must reference a declared entity primitive
- [update-record](./update-record.md) — often follows a `create_record` to set additional fields once an ID is known
- [assign](./assign.md) — newly created work items are typically routed immediately after creation
- [create-obligation](./create-obligation.md) — when the new record represents a required duty rather than a general task, prefer `create_obligation`
- [event](../event/event.md) — emitting an event alongside `create_record` signals other services that the new instance exists

## Example

```yaml
primitive: create_record
entity: Task
values:
  type: CollectionFollowUp
  owner: CollectionsQueue
  related_invoice: invoice.id
  due_at: now + 2 business_days
```

```yaml
primitive: create_record
entity: CreditMemo
values:
  order_id: order.id
  customer_id: order.customer_id
  amount: return.approved_amount
  reason: return.reason_code
  issued_at: now
```
