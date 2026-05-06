# Reversal

Undoes a specific applied event or transaction, restoring the prior state. More targeted than compensation — it addresses a single prior effect rather than an entire workflow. A reversal is always tied to an original event and produces a traceable record that links the undo to what it reversed.

## Shape

```yaml
primitive: reversal
original_event: PaymentApplied
effects:
  - remove_payment_application
  - reopen_invoice_balance
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `original_event` | event name | yes | The name of the event whose effects are being undone |
| `effects` | ordered list of strings | yes | The specific actions to execute to undo the original event, in the order they should run |
| `requires` | list of conditions | no | Authority or state conditions that must hold before the reversal can be applied |
| `emit` | emitted event reference | no | An event to publish after the reversal is complete |
| `audit_reason` | string | no | A required justification captured at the time of reversal for audit records |
| `linked_to` | event reference | no | The specific instance of the original event being reversed (e.g. a payment transaction ID) |

## Use when

- A single transaction was applied incorrectly and needs to be undone without unwinding an entire workflow
- A payment, credit, or adjustment was posted in error and the accounting record must be corrected
- A state transition was applied to an entity and needs to be reversed after a dispute or correction
- You need a traceable undo that preserves the original event in the audit log while marking it as reversed

## Composes with

- [compensation](./compensation.md) — compensation orchestrates multiple reversals in sequence; reversal is the unit that compensation calls
- [event](../event/event.md) — a reversal always references the original event by name and ideally by instance ID
- [emitted-event](../event/emitted-event.md) — the reversal emits its own event (e.g. `PaymentReversed`) that becomes a source event for downstream rules
- [override](./override.md) — an authorized reversal may require an override-style authority check when the original effect was sanctioned

## Example

```yaml
primitive: reversal
original_event: PaymentApplied
effects:
  - remove_payment_application
  - reopen_invoice_balance
linked_to: payment.transaction_id
requires:
  - actor.role in [BillingManager, FinanceOps]
audit_reason: "Payment applied to wrong invoice; reversing to allow correct reapplication."
emit:
  name: PaymentReversed
  payload:
    payment_id: payment.id
    original_invoice_id: invoice.id
    reversed_at: now
    reversed_by: actor.id
```

```yaml
primitive: reversal
original_event: CreditApplied
effects:
  - remove_credit_memo_application
  - restore_credit_memo_balance
linked_to: credit_memo.id
requires:
  - actor.role = AccountingManager
audit_reason: "Credit memo applied in error during month-end close; reversing per finance review."
emit:
  name: CreditReversed
  payload:
    credit_memo_id: credit_memo.id
    account_id: account.id
```
