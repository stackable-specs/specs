# Compensation

Reverses prior effects when a later step in a workflow fails. Used in saga-style workflows to maintain consistency across distributed operations that cannot be wrapped in a single atomic transaction. Compensation is not error handling — it is a deliberate, ordered rollback of effects that have already been committed.

## Shape

```yaml
primitive: compensation
when: shipment_creation_failed
reverse:
  - inventory_reservation
  - payment_authorization
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `when` | event name or condition | yes | The failure event or condition that triggers compensation to execute |
| `reverse` | ordered list of action names | yes | The effects to undo, listed in reverse order of application (last applied is first reversed) |
| `emit` | emitted event reference | no | An event to publish after compensation completes, signaling that rollback is done |
| `timeout` | duration | no | Maximum time allowed for compensation to complete before an alert is raised |
| `on_compensation_failed` | string or outcome | no | What to do if one of the reversal steps itself fails (e.g. escalate to manual intervention) |

## Use when

- A multi-step workflow has committed side effects in step N and step N+1 fails, requiring step N to be undone
- You are coordinating across services (billing, inventory, shipping) that each have their own state and no shared transaction boundary
- A business operation must be all-or-nothing even though it spans distributed systems
- You need a documented rollback plan that is part of the rule specification, not implicit in application code

## Composes with

- [reversal](./reversal.md) — each entry in the `reverse` list is typically implemented as a reversal primitive targeting a specific prior effect
- [retry](./retry.md) — retries are exhausted before compensation begins; compensation is the final fallback when retries fail
- [emitted-event](../event/emitted-event.md) — emit a `CompensationCompleted` or `WorkflowRolledBack` event for audit and downstream awareness
- [exception](./exception.md) — certain exceptions (e.g. fraud detection) may bypass retries and trigger compensation immediately

## Example

```yaml
primitive: compensation
when: shipment_creation_failed
reverse:
  - inventory_reservation
  - payment_authorization
emit:
  name: OrderCompensationCompleted
  payload:
    order_id: order.id
    reversed_steps:
      - inventory_reservation
      - payment_authorization
    compensated_at: now
timeout: 10 minutes
on_compensation_failed: escalate_to_operations_team
```

```yaml
primitive: compensation
when: contract_activation_failed
reverse:
  - provisioned_entitlements
  - applied_first_invoice_discount
  - updated_account_tier
emit:
  name: ContractActivationRolledBack
  payload:
    contract_id: contract.id
    account_id: account.id
```
