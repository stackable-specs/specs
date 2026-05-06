# Missing Fact

Defines what happens when a required fact is unavailable at decision time. Without this primitive, missing data causes silent failures or undefined behavior. Declaring the missing-fact response makes that behavior explicit and testable.

## Shape

```yaml
primitive: missing_fact
fact: <field_path>
effect:
  decision: <decision_value>
  reason_code: <reason_code>
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `fact` | field path | yes | The fact that may be absent |
| `effect.decision` | decision value | yes | The outcome to produce when the fact is missing (e.g. `NeedsReview`, `Ineligible`, `Blocked`) |
| `effect.reason_code` | string | yes | The reason code emitted so downstream systems and humans know why the fallback was triggered |

## Use when

- A rule marks a fact as `required: true` and needs a defined response when it is absent
- Data may legitimately be missing at decision time (new customers, incomplete onboarding, enrichment failures)
- Defaulting to a hardcoded value would be silently wrong — you want the absence to be visible
- Audit or compliance requires that missing-data decisions be distinguishable from normal outcomes

## Composes with

- [fact](./fact.md) — the fact being guarded
- [fact-source](./fact-source.md) — a stale or untrusted source can be treated as missing
- [fallback](../exception/fallback.md) — the broader pattern for when no rule produces a result; missing-fact is a data-level version
- [reason-code](../explanation/reason-code.md) — the effect always emits a reason code
- [enum-decision](../decision/enum-decision.md) — the effect typically resolves to a specific enum value

## Example

```yaml
primitive: missing_fact
fact: customer.tax_id
effect:
  decision: NeedsReview
  reason_code: MissingTaxId
```

```yaml
primitive: missing_fact
fact: order.delivered_at
effect:
  decision: Ineligible
  reason_code: DeliveryNotConfirmed
```
