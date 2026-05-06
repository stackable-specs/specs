# Override

Replaces a base outcome with an authorized alternative. An override requires authority and must be auditable — it is not a silent suppression but a named substitution that records who authorized it, why, and what changed. Overrides are the mechanism by which human judgment supersedes automated rule outcomes.

## Shape

```yaml
primitive: override
base_outcome: credit_hold = true
override_outcome: credit_hold = false
requires:
  - approver.role = VPFinance
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `base_outcome` | outcome expression | yes | The outcome produced by the base rule that this override replaces |
| `override_outcome` | outcome expression | yes | The alternative outcome to apply in place of the base |
| `requires` | list of conditions | yes | Authority conditions that must hold for the override to be valid; typically role or permission checks |
| `reason` | string | no | A free-text justification recorded in the audit log when the override is applied |
| `expires_at` | date or duration | no | If set, the override is automatically revoked after this point and the base rule resumes |
| `scope` | string | no | Limits the override to a specific entity instance, segment, or time window |

## Use when

- A manager or senior stakeholder must be able to reverse an automated decision for a specific account or transaction
- A rule outcome is correct in general but wrong for a specific case that has been escalated and reviewed
- Regulatory compliance requires a documented record of who authorized a deviation from standard policy
- A temporary commercial agreement changes the outcome for one customer without altering the underlying rule logic

## Composes with

- [exception](./exception.md) — an exception identifies that a deviation applies; an override defines what the replacement outcome is and who can authorize it
- [waiver](./waiver.md) — a waiver is a specialized override focused on forgiving a fee or obligation
- [compensation](./compensation.md) — if an override is later revoked, a compensation may be needed to reverse effects already applied
- [governance](../governance/approval.md) — the `requires` authority chain typically references a governance approval primitive

## Example

```yaml
primitive: override
base_outcome: credit_hold = true
override_outcome: credit_hold = false
requires:
  - approver.role = VPFinance
reason: "Account has signed a new multi-year contract; credit hold waived pending payment plan."
expires_at: 30 days
scope: account.id = ACC-10042
```

```yaml
primitive: override
base_outcome: invoice.collection_status = Escalated
override_outcome: invoice.collection_status = PaymentPlanActive
requires:
  - approver.role in [CollectionsManager, VPFinance]
  - payment_plan.signed = true
reason: "Customer entered a signed payment plan agreement."
```
