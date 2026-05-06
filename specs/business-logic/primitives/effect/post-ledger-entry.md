# Post Ledger Entry

Records a balanced financial debit/credit pair to the accounting ledger. Every posting must balance — the debit account and credit account each receive the same amount, preserving double-entry integrity. The primitive captures the accounting intent of a business event without specifying how the general ledger is physically stored; the ledger service owns persistence and reconciliation.

## Shape

```yaml
primitive: post_ledger_entry
debit: AccountsReceivable
credit: Revenue
amount: invoice.total
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `debit` | string | yes | Name of the ledger account to debit (increase for asset/expense accounts, decrease for liability/equity/revenue accounts) |
| `credit` | string | yes | Name of the ledger account to credit (increase for liability/equity/revenue accounts, decrease for asset/expense accounts) |
| `amount` | number or field path | yes | The monetary amount of the posting; must resolve to a positive number |
| `currency` | string or field path | no | ISO 4217 currency code (e.g., `USD`, `EUR`); defaults to the entity's functional currency if omitted |
| `reference` | field path | no | The business document that originated the posting (e.g., `invoice.id`, `payment.id`, `credit_memo.id`) |
| `effective_date` | date or field path | no | The accounting date of the entry; defaults to `today` if omitted; use `invoice.issued_at` or `payment.settled_at` for period accuracy |
| `memo` | string | no | Human-readable description attached to the journal entry for reconciliation |

## Use when

- A business event has financial significance and must be captured in the general ledger (e.g., recognizing revenue when an invoice is issued, recording a payment receipt)
- A rule outcome reverses a prior posting (e.g., debit `Revenue` and credit `AccountsReceivable` to void an invoice)
- Accruals, deferrals, or adjustments must be posted on a schedule driven by business logic
- You need a clean boundary between business rule logic and accounting mechanics — the rule declares what to post; the ledger service handles chart-of-accounts validation and period close
- Regulatory or audit requirements demand that every financial state change have a corresponding immutable ledger record

## Composes with

- [event](../event/event.md) — `InvoiceIssued`, `PaymentReceived`, `RefundProcessed` events are the canonical triggers for ledger postings
- [transition](../state/transition.md) — transitions from `Draft` to `Issued` or `Pending` to `Settled` often co-emit a `post_ledger_entry`
- [release-resource](./release-resource.md) — financial credit reserves released on payment confirmation pair with a corresponding reversal ledger entry
- [create-record](./create-record.md) — a `JournalEntry` record may be created alongside the `post_ledger_entry` for in-system traceability
- [fact](../fact/fact.md) — the `amount` field commonly references a derived fact (e.g., `invoice.tax_adjusted_total`) computed earlier in the rule chain

## Example

```yaml
primitive: post_ledger_entry
debit: AccountsReceivable
credit: Revenue
amount: invoice.total
currency: invoice.currency
reference: invoice.id
effective_date: invoice.issued_at
memo: Revenue recognition on invoice issuance
```

```yaml
primitive: post_ledger_entry
debit: Cash
credit: AccountsReceivable
amount: payment.amount
currency: payment.currency
reference: payment.id
effective_date: payment.settled_at
memo: Payment receipt applied to outstanding invoice
```
