# Grant Entitlement

Gives a subject access to a named feature, service tier, or capability for a defined effective period. Entitlements are the positive grants in a feature-access model — downstream systems check whether an entitlement exists and is currently in period before allowing access. The rule declares the grant; the entitlement service enforces it at the boundary.

## Shape

```yaml
primitive: grant_entitlement
subject: account
entitlement: AdvancedAnalytics
effective_period: subscription.current_period
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `subject` | entity reference or field path | yes | The entity receiving the entitlement (e.g., `account`, `user`, `organization`) |
| `entitlement` | string | yes | Pascal-case name of the feature or capability being granted |
| `effective_period` | period reference or expression | yes | The time window during which the entitlement is active; may reference a subscription period, a literal range, or a duration from `now` |
| `granted_by` | string | no | Name of the rule or policy that issued the grant, for audit purposes |
| `metadata` | map | no | Optional key-value pairs scoping the grant (e.g., seat count, usage limit) |

## Use when

- A subscription purchase or upgrade should unlock a feature for the duration of the billing period
- A trial or promotional offer provides access to a capability for a defined time window
- An approval workflow concludes and the approved party gains access to a restricted feature
- Entitlement state must be queryable by downstream services without those services containing business rule logic
- You need an auditable, time-bounded record of who was granted access to what and when

## Composes with

- [revoke-entitlement](./revoke-entitlement.md) — the mirror effect; cancellation, downgrade, or expiry rules emit `revoke_entitlement` against previously granted entitlements
- [allow-action](./allow-action.md) — `grant_entitlement` governs feature access; `allow_action` governs discrete operation permissions; both are often needed together
- [entity](../entity/entity.md) — the `subject` must be a declared entity so the entitlement can be looked up by the subject's ID at enforcement time
- [event](../event/event.md) — a `SubscriptionUpgraded` or `TrialStarted` event commonly triggers this effect
- [transition](../state/transition.md) — state transitions that move a subscription from `Trialing` to `Active` typically emit a `grant_entitlement` as a co-effect

## Example

```yaml
primitive: grant_entitlement
subject: account
entitlement: AdvancedAnalytics
effective_period: subscription.current_period
granted_by: SubscriptionActivationRule
```

```yaml
primitive: grant_entitlement
subject: account
entitlement: PrioritySupport
effective_period:
  starts_at: now
  ends_at: now + 30 days
granted_by: TrialUpgradePromotion
metadata:
  seat_limit: 5
```
