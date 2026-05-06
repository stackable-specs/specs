# Enum Decision

A rule that resolves to one value from a fixed, named set of outcomes. Enum decisions model business judgments that have more than two possible results — the full outcome space is declared upfront so consumers know every state the decision can produce.

## Shape

```yaml
primitive: enum_decision
name: eligibility_status
values:
  - Eligible
  - Ineligible
  - NeedsReview
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Snake-case name of the decision, phrased as a noun or noun phrase (e.g. `eligibility_status`, `approval_tier`) |
| `values` | string list | yes | The exhaustive set of named outcomes the decision can produce; order is significant — first match wins when used with a classification |

## Use when

- The outcome space has three or more distinct, named states that carry different business meaning
- Downstream routing, workflow branching, or UI display depends on knowing which named state was reached
- You want to prevent ad-hoc string values from proliferating across rules that share the same decision concept
- The decision will be paired with a `classification` primitive that maps conditions to each value
- Historical records need a stable, human-readable status rather than a boolean or a raw score

## Composes with

- [classification](./classification.md) — the classification primitive maps conditions to each enum value; enum-decision declares the value space, classification populates it
- [boolean-decision](./boolean-decision.md) — a boolean decision is a specialization of an enum decision with exactly two values (`true` / `false`)
- [reasoned-decision](./reasoned-decision.md) — attach reason codes to explain which value was selected and why
- [fact](../fact/fact.md) — facts carry the field values that conditions reference when producing an enum result

## Example

```yaml
primitive: enum_decision
name: eligibility_status
values:
  - Eligible
  - Ineligible
  - NeedsReview
```

```yaml
primitive: enum_decision
name: claim_disposition
values:
  - Approved
  - Denied
  - PendingDocumentation
  - Escalated
```
