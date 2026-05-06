# Rollout Control

Gradually releases a rule change to a percentage of traffic, with a stable key for consistent assignment. Rollout control lets teams validate new rule behavior in production against real data before committing to a full cutover, with the assignment deterministic so any given entity always sees the same version.

## Shape

```yaml
primitive: rollout_control
strategy: percentage
percentage: 10
stable_key: customer.id
fallback_rule_version: pricing_v1
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `strategy` | enum | yes | Method used to determine which requests see the new version. One of: `percentage`, `cohort`, `feature_flag`, `allowlist` |
| `target_rule_version` | string | no | The new rule version being rolled out; defaults to the current latest version if omitted |
| `fallback_rule_version` | string | yes | The stable version to use for traffic not included in the rollout |
| `percentage` | integer | no | Required when `strategy` is `percentage`. Portion of traffic (0–100) directed to the new version |
| `stable_key` | field path | no | The fact path used to hash-assign an entity consistently to the same side of the rollout (e.g., `customer.id`) |
| `cohort` | string | no | Required when `strategy` is `cohort`. Named segment identifier (e.g., `beta_testers`, `us_west`) |
| `feature_flag` | string | no | Required when `strategy` is `feature_flag`. The flag key evaluated at runtime to route traffic |
| `allowlist` | string list | no | Required when `strategy` is `allowlist`. Explicit list of entity IDs that receive the new version |
| `start_date` | date | no | Date from which the rollout is active; traffic before this date uses the fallback |
| `end_date` | date | no | Date on which the rollout concludes and all traffic is expected to move to the new version |

## Use when

- A rule change has passed review but carries enough uncertainty that a partial production test is warranted before full rollout
- You need to measure the business impact of a rule change (e.g., refund rate, revenue delta) on a controlled slice of traffic
- A rule update affects a large user population and a phased migration is safer than a big-bang cutover
- You want assignment to be sticky so that a single customer consistently sees the same rule version across repeated requests
- The new rule version should only be tested with a specific named cohort (e.g., beta customers, a single geography)

## Composes with

- [rule-version](./rule-version.md) — rollout control targets specific named rule versions; rule versioning makes the `target_rule_version` and `fallback_rule_version` unambiguous
- [kill-switch](./kill-switch.md) — if the rollout produces unexpected results, a kill switch can immediately halt it and revert all traffic to the fallback without a deployment
- [../observability/metric](../observability/metric.md) — metrics computed per rule version show whether the rollout target is outperforming, matching, or underperforming the fallback
- [../observability/drift-monitor](../observability/drift-monitor.md) — drift monitors can be scoped to the rollout cohort to detect unexpected behavioral shifts early

## Example

```yaml
primitive: rollout_control
strategy: percentage
percentage: 10
stable_key: customer.id
target_rule_version: pricing_v2
fallback_rule_version: pricing_v1
start_date: 2026-04-01
end_date: 2026-04-30
```

```yaml
primitive: rollout_control
strategy: cohort
cohort: enterprise_beta
target_rule_version: refund_eligibility_v4
fallback_rule_version: refund_eligibility_v3
stable_key: account.id
```
