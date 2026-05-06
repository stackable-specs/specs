# Waiver

Forgives a fee, requirement, or obligation. A waiver sets the item to zero or removes it entirely, and always requires a reason. Waivers are a distinct concept from overrides because they concern specific financial or contractual obligations rather than general rule outcomes — they are subject to revenue and audit scrutiny.

## Shape

```yaml
primitive: waiver
waived_item: late_fee
waived_amount: late_fee.amount
reason_required: true
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `waived_item` | string | yes | The fee, charge, obligation, or requirement being forgiven |
| `waived_amount` | expression or `full` | yes | The amount forgiven; use `full` to remove the entire item, or an expression for partial waivers |
| `reason_required` | boolean | yes | Whether a human-supplied reason must be recorded at the time of waiver |
| `reason` | string | no | The captured justification; mandatory if `reason_required` is true |
| `requires` | list of conditions | no | Authority conditions the actor applying the waiver must satisfy |
| `max_per_period` | integer + duration | no | Limits how many times this waiver can be applied to the same entity within a time window |
| `expires_at` | date | no | Date after which the waiver is no longer available without re-authorization |

## Use when

- A late fee, interest charge, or penalty should be forgiven for a specific customer or invoice
- A mandatory requirement (e.g. proof-of-insurance, certification) is being temporarily excused for a documented reason
- Goodwill adjustments for high-value or long-tenured customers need to be tracked and capped
- Regulatory or contractual terms entitle a customer class to fee forgiveness under defined conditions

## Composes with

- [exception](./exception.md) — an exception can automatically trigger a waiver for a qualifying customer segment without manual intervention
- [override](./override.md) — a waiver is a specialized override; when the forgiven item is part of a larger outcome, an override wraps the waiver
- [explanation](../explanation/rationale.md) — the mandatory reason field feeds directly into the explanation primitive for audit output
- [governance](../governance/approval.md) — high-value waivers typically require approval from a senior role before being applied

## Example

```yaml
primitive: waiver
waived_item: late_fee
waived_amount: full
reason_required: true
reason: "First-time late payment; customer has 5-year tenure with no prior delinquency."
requires:
  - actor.role in [AccountManager, CollectionsManager]
max_per_period: 1 per 12 months
```

```yaml
primitive: waiver
waived_item: early_termination_fee
waived_amount: early_termination_fee.amount * 0.5
reason_required: true
requires:
  - approver.role = VPSales
  - contract.remaining_months <= 3
reason: "Partial waiver granted as part of renewal negotiation."
```
