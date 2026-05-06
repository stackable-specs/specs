# Approval Requirement

Requires human sign-off before an outcome takes effect, with a defined approver role and the conditions under which approval is needed. Approval requirements encode organizational authorization controls directly into business rules, preventing high-risk outcomes from executing automatically without the appropriate oversight.

## Shape

```yaml
primitive: approval_requirement
required_when:
  - refund.amount > 500
approver_role: FinanceManager
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `required_when` | predicate list | yes | One or more conditions that, when true, gate the outcome behind an approval |
| `approver_role` | string | yes | The organizational role authorized to approve; must match an identity system role |
| `timeout_hours` | integer | no | How long to wait for approval before escalating or auto-rejecting |
| `escalation_role` | string | no | Role to notify if the primary approver does not respond within `timeout_hours` |
| `auto_reject_on_timeout` | boolean | no | If true, the outcome is denied when the approval window expires without a response |
| `instructions` | string | no | Human-readable context shown to the approver to help them make a decision |

## Use when

- An outcome carries financial, legal, or reputational risk above a defined threshold
- Regulatory or contractual compliance requires documented human authorization for specific actions
- You want to enforce a four-eyes principle without hard-coding it in application logic
- Approvals are needed only for a subset of cases and the condition is expressible as a predicate
- You need an audit trail showing that a specific role authorized a specific outcome

## Composes with

- [audit-requirement](./audit-requirement.md) — approval decisions (granted, denied, timed out) must be logged alongside the rule outcome they govern
- [policy-reference](./policy-reference.md) — the approval threshold is typically mandated by a contract or policy; reference it to justify the requirement
- [../explanation/reason-code](../explanation/reason-code.md) — when an outcome is blocked pending approval, a reason code communicates this state to the caller
- [../composition/rule](../composition/rule.md) — attach an approval requirement to a rule to gate its effects on human authorization

## Example

```yaml
primitive: approval_requirement
required_when:
  - refund.amount > 500
approver_role: FinanceManager
timeout_hours: 24
escalation_role: FinanceDirector
auto_reject_on_timeout: false
instructions: "Review the refund request and attached order history before approving."
```

```yaml
primitive: approval_requirement
required_when:
  - account.tier == "enterprise"
  - contract_change.type == "termination"
approver_role: AccountExecutive
timeout_hours: 48
escalation_role: VP_Sales
auto_reject_on_timeout: true
```
