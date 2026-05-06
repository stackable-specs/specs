# Effective Period

The bounded interval during which a policy, price, rule, or agreement is considered active and applicable. Effective periods make temporally-scoped business logic explicit — they allow rules and facts to be versioned by time rather than overwritten in place.

## Shape

```yaml
primitive: effective_period
effective_from: policy.effective_from
effective_to: policy.effective_to
active_when: effective_from <= as_of_date < effective_to
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `effective_from` | field path | yes | The instant from which the record becomes active (inclusive) |
| `effective_to` | field path | yes | The instant at which the record ceases to be active (exclusive) |
| `active_when` | expression | yes | The logical predicate that evaluates whether the record is active for a given `as_of_date` |

## Use when

- A price, policy, tax rate, or configuration must be versioned over time without losing history
- Rules must evaluate which version of a fact was active at the moment a business event occurred
- A contract, agreement, or entitlement has a defined start and end date that governs its applicability
- Multiple overlapping versions of a rule or configuration must be prevented or detected
- An audit requires knowing exactly which policy governed a decision at a specific point in time

## Composes with

- [as-of-evaluation](./as-of-evaluation.md) — as-of evaluation selects the version of a fact or rule whose effective period covers the as-of date
- [instant](./instant.md) — `effective_from` and `effective_to` are named instants on the entity
- [window](./window.md) — an effective period is a window expressed in terms of validity rather than eligibility
- [schedule](./schedule.md) — schedules that drive recurring events must check which policy effective period covers the run date
- [fact](../fact/fact.md) — facts sourced from versioned entities should carry the effective period to support point-in-time lookup

## Example

```yaml
primitive: effective_period
effective_from: policy.effective_from
effective_to: policy.effective_to
active_when: effective_from <= as_of_date < effective_to
```

```yaml
primitive: effective_period
effective_from: price.valid_from
effective_to: price.valid_to
active_when: effective_from <= order.submitted_at < effective_to
```
