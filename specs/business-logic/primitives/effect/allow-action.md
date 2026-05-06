# Allow Action

Explicitly permits a named action to proceed. Used in default-deny policy architectures where every action requires a positive grant before it can execute. Without a matching `allow_action` in scope, the action is rejected; the allow effect is the signal that all required conditions have been satisfied and the gate should open.

## Shape

```yaml
primitive: allow_action
action: ApproveRefund
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `action` | string | yes | Pascal-case name of the action being permitted |
| `granted_by` | string | no | Identifier of the role, rule, or policy that issued the grant, for audit purposes |
| `expires_at` | timestamp or expression | no | If set, the permission lapses after this time and must be re-evaluated |
| `scope` | map | no | Optional constraints scoping the permission (e.g., maximum amount, specific entity ID) |

## Use when

- Your policy model is default-deny and every executable action must be explicitly unlocked by a rule
- An approval workflow has completed and the downstream system needs a structured signal to proceed (e.g., allow a refund after a supervisor has approved it)
- A conditional permission must be granted for a bounded window of time (e.g., allow order placement for a trial account for 30 days)
- You want to separate the decision to permit an action from the mechanics of executing it, keeping the enforcement layer clean
- Multiple independent rules each contribute partial grants that must all be present before the action is allowed

## Composes with

- [block-action](./block-action.md) — `allow_action` and `block_action` are the two poles of action-gate enforcement; a conflict resolution strategy determines precedence
- [grant-entitlement](./grant-entitlement.md) — entitlements grant access to features; `allow_action` grants permission for discrete operations
- [decision](../decision/boolean-decision.md) — the upstream boolean decision determines whether the `allow_action` effect fires
- [condition](../condition/predicate.md) — conditions guard when the allow is issued; `allow_action` itself carries no condition logic
- [event](../event/event.md) — an approval event commonly triggers an `allow_action` effect that persists until the permitted window closes

## Example

```yaml
primitive: allow_action
action: ApproveRefund
granted_by: SupervisorApprovalRule
scope:
  max_amount: 500.00
  order_id: order.id
```

```yaml
primitive: allow_action
action: PlaceOrder
granted_by: TrialAccountPolicy
expires_at: account.trial_ends_at
```
