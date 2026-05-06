# Rule Version

Tracks changes to a rule over time, enabling as-of evaluation and migration control. Rule versioning ensures that historical decisions can be reproduced using the rule logic that was active at the time they were made, and that rule upgrades are applied deliberately rather than silently.

## Shape

```yaml
primitive: rule_version
rule_id: refund_eligibility
version: 3
effective_from: 2026-01-01
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `rule_id` | string | yes | Stable identifier for the rule across all versions |
| `version` | integer | yes | Monotonically increasing version number |
| `effective_from` | date | yes | The date from which this version is the active one |
| `effective_until` | date | no | The date after which this version is superseded; omit for the current version |
| `supersedes` | integer | no | Version number this one replaces |
| `change_summary` | string | no | Human-readable description of what changed and why |
| `author` | string | no | Identity of the person or system that published this version |

## Use when

- A rule's logic changes and historical evaluations must remain reproducible against the rule that was in effect at the time
- You are running an audit and need to replay decisions as they would have been made on a past date
- A rule change is under review and you want to stage the new version before making it effective
- Multiple rule versions need to co-exist during a phased rollout
- You need to roll back to a prior rule version without a code deployment

## Composes with

- [policy-reference](./policy-reference.md) — each version should reference the policy document that motivated it, so the version history doubles as a change log with business justification
- [rollout-control](./rollout-control.md) — rollout control targets specific rule versions, directing a percentage of traffic to the new version before full promotion
- [kill-switch](./kill-switch.md) — a kill switch references a specific version to fall back to; rule versioning makes that target unambiguous
- [../composition/rule](../composition/rule.md) — a rule carries a rule_version so evaluators know which logic applies

## Example

```yaml
primitive: rule_version
rule_id: refund_eligibility
version: 3
effective_from: 2026-01-01
supersedes: 2
change_summary: "Extended return window from 30 to 45 days for loyalty tier Gold and above."
author: product.policy@example.com
```

```yaml
primitive: rule_version
rule_id: pricing_v2
version: 2
effective_from: 2026-03-15
effective_until: 2026-06-30
supersedes: 1
change_summary: "Adjusted promotional discount cap from 20% to 15% per finance review."
```
