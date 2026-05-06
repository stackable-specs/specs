# Revoke Entitlement

Removes a subject's access to a previously granted feature or service. Revocation is immediate and explicit — the entitlement record is terminated rather than expired, and a reason code is attached so that the subject and downstream systems can understand why access was withdrawn. Use it whenever a business rule must actively end access rather than letting a period naturally lapse.

## Shape

```yaml
primitive: revoke_entitlement
subject: account
entitlement: AdvancedAnalytics
reason_code: SubscriptionCancelled
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `subject` | entity reference or field path | yes | The entity whose entitlement is being removed |
| `entitlement` | string | yes | Pascal-case name of the feature or capability being revoked |
| `reason_code` | string | yes | Machine-readable code explaining why access was withdrawn (e.g., `SubscriptionCancelled`, `PaymentFailure`, `PlanDowngrade`, `AccountSuspended`) |
| `effective_at` | timestamp or expression | no | When the revocation takes effect; defaults to `now` if omitted; may be set to a future timestamp for grace-period revocations |
| `revoked_by` | string | no | Name of the rule or policy that issued the revocation, for audit purposes |

## Use when

- A subscription is cancelled, downgraded, or lapses and the associated features should be withdrawn
- An account is suspended or placed on credit hold and feature access must be restricted immediately
- A trial period ends and the trial-only capabilities should no longer be accessible
- A compliance or policy violation requires immediate revocation of a specific capability
- You need a structured, auditable record that access was intentionally withdrawn (as opposed to simply expiring)

## Composes with

- [grant-entitlement](./grant-entitlement.md) — the mirror effect; every revocable entitlement was initially created by a `grant_entitlement`
- [block-action](./block-action.md) — pair with `block_action` when feature withdrawal must also prevent specific operations from executing
- [notify](./notify.md) — notify the subject that access has been revoked and explain what they have lost and how to restore it
- [transition](../state/transition.md) — transitions to terminal or suspended states (e.g., `Active` → `Cancelled`) typically emit `revoke_entitlement` as a co-effect
- [event](../event/event.md) — a `SubscriptionCancelled` or `PaymentFailed` event is the common trigger; the event carries the reason context

## Example

```yaml
primitive: revoke_entitlement
subject: account
entitlement: AdvancedAnalytics
reason_code: SubscriptionCancelled
effective_at: subscription.cancelled_at
revoked_by: SubscriptionCancellationRule
```

```yaml
primitive: revoke_entitlement
subject: account
entitlement: PrioritySupport
reason_code: PlanDowngrade
effective_at: now + 30 days
revoked_by: PlanDowngradePolicy
```
