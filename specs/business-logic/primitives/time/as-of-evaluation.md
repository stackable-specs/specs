# As-Of Evaluation

A named evaluation context that pins all fact lookups, rule selections, and price retrievals to a specific point in time. As-of evaluation ensures that a decision is reproducible — re-running it with the same `as_of` date produces the same result regardless of subsequent data changes.

## Shape

```yaml
primitive: as_of_evaluation
as_of: decision_time
use:
  - facts_as_of
  - rules_as_of
  - prices_as_of
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `as_of` | instant or field path | yes | The point in time that all lookups in this evaluation context are pinned to |
| `use` | list of strings | yes | The categories of data that must be resolved as of the named instant — e.g., `facts_as_of`, `rules_as_of`, `prices_as_of`, `rates_as_of` |

## Use when

- A decision engine must reproduce a prior decision using the facts and rules that were current at the original decision time
- Regulatory audits require demonstrating which rule version governed a specific transaction
- A price or tax rate is effective-dated and must be selected based on when an order was placed, not when it is processed
- Backdated corrections must be evaluated against the rules that were active on the original transaction date
- Simulation or what-if analysis requires evaluating a scenario as if it is a past or future date

## Composes with

- [effective-period](./effective-period.md) — as-of evaluation selects the version of any record whose effective period covers the `as_of` instant
- [instant](./instant.md) — the `as_of` value is an instant, either hardcoded or sourced from an entity field
- [window](./window.md) — as-of evaluation checks whether the `as_of` date falls within a given window
- [fact](../fact/fact.md) — facts are the primary target of as-of resolution; each fact is looked up at the `as_of` timestamp
- [condition](../condition/condition.md) — conditions evaluated within an as-of context use the pinned snapshots of all referenced facts

## Example

```yaml
primitive: as_of_evaluation
as_of: decision_time
use:
  - facts_as_of
  - rules_as_of
  - prices_as_of
```

```yaml
primitive: as_of_evaluation
as_of: order.submitted_at
use:
  - prices_as_of
  - tax_rates_as_of
  - discount_rules_as_of
```
