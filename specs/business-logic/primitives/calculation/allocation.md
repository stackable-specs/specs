# Allocation

Distributes a single amount across a set of recipient records according to a defined basis. Allocation encodes the policy that governs how a payment, credit, or charge is spread — oldest-first, proportional, or by priority — so that distribution logic is explicit, auditable, and consistent.

## Shape

```yaml
primitive: allocation
amount: payment.amount
recipients: open_invoices
basis: oldest_due_date_first
target: payment_applications
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `amount` | field path or number | yes | The total amount to distribute across recipients |
| `recipients` | collection name or field path | yes | The set of records that will receive a portion of the amount |
| `basis` | enum | yes | The rule governing how the amount is divided: `oldest_due_date_first`, `proportional`, `equal`, `priority_order`, `fixed_percentage` |
| `target` | field path | yes | The collection of allocation records produced as output |
| `percentages` | map of id → decimal | no | Required when `basis` is `fixed_percentage`; maps each recipient identifier to its share |

**Basis options:**
- `oldest_due_date_first` — fill earliest-due recipients to zero before moving to later ones
- `proportional` — each recipient receives a share proportional to its outstanding balance
- `equal` — divide evenly; remainder goes to the first recipient
- `priority_order` — recipients carry an explicit priority rank; highest rank is funded first
- `fixed_percentage` — each recipient receives a pre-specified percentage of the total amount

## Use when

- A payment, credit memo, or adjustment must be applied across multiple open invoices or charges
- The allocation policy differs by customer agreement, payment method, or collection strategy
- You need a traceable record showing how each dollar of a payment was applied
- Partial payments require a defined rule for which invoices absorb the shortfall

## Composes with

- [aggregation](./aggregation.md) — aggregate open balances to validate that the amount does not exceed total owed
- [entity/collection](../entity/collection.md) — defines and filters the recipient set before allocation runs
- [formula](./formula.md) — compute proportional shares or remaining balances within the allocation
- [rounding](./rounding.md) — round individual allocation amounts and handle remainder distribution
- [fact](../fact/fact.md) — recipient priority, due dates, and balances are sourced as facts

## Example

```yaml
primitive: allocation
amount: payment.amount
recipients: open_invoices
basis: oldest_due_date_first
target: payment_applications
```

```yaml
primitive: allocation
amount: credit_memo.amount
recipients: open_charges
basis: proportional
target: credit_applications
```
