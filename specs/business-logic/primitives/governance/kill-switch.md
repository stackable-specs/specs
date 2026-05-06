# Kill Switch

Disables a rule immediately and falls back to a safe prior version, without a deployment. A kill switch is a named, pre-configured circuit breaker that an operator can flip in production to neutralize a rule that is causing harm — bad pricing, runaway refunds, a broken eligibility check — within seconds.

## Shape

```yaml
primitive: kill_switch
target: pricing_v2
when_enabled:
  disable: pricing_v2
  fallback: pricing_v1
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `target` | string | yes | The rule version identifier that this kill switch disables when activated |
| `when_enabled.disable` | string | yes | The rule version to stop routing traffic to upon activation |
| `when_enabled.fallback` | string | yes | The rule version to route all traffic to once the switch is enabled |
| `owner` | string | no | Role or identity authorized to activate and deactivate this kill switch |
| `notify` | string list | no | Roles or channels to alert when the kill switch is activated or deactivated |
| `requires_approval` | boolean | no | If true, activation requires authorization from the `owner` before taking effect; defaults to false for break-glass scenarios |
| `auto_deactivate_after` | duration | no | Duration after which the kill switch automatically deactivates if not manually extended (e.g., `4 hours`) |

## Use when

- A newly released rule version is producing incorrect or harmful outcomes in production and must be halted immediately
- You want to pre-configure a rollback path during rule deployment so operators do not have to improvise under pressure
- A rule change is deployed alongside a kill switch as a safety contract: ship fast, revert faster
- You need to give non-engineers (product managers, compliance officers) the ability to disable a rule without a code change
- A feature flag or rollout control is insufficient because you need a hard stop, not a traffic shift

## Composes with

- [rule-version](./rule-version.md) — both `target` and `fallback` reference specific rule versions; rule versioning makes the kill switch's behavior deterministic and unambiguous
- [rollout-control](./rollout-control.md) — rollout control and kill switches are complementary safety mechanisms: rollout control limits blast radius, kill switch removes it entirely
- [../observability/alert](../observability/alert.md) — alerts are the trigger that tells an operator a kill switch needs to be activated; wire them together in runbooks
- [audit-requirement](./audit-requirement.md) — kill switch activation is a high-stakes event and must be logged with actor, timestamp, and reason

## Example

```yaml
primitive: kill_switch
target: pricing_v2
when_enabled:
  disable: pricing_v2
  fallback: pricing_v1
owner: EngineeringOnCall
notify:
  - engineering_oncall_channel
  - product_policy_owner
auto_deactivate_after: 4 hours
```

```yaml
primitive: kill_switch
target: refund_eligibility_v5
when_enabled:
  disable: refund_eligibility_v5
  fallback: refund_eligibility_v4
owner: FinanceManager
requires_approval: false
notify:
  - finance_ops_channel
  - compliance_team
```
