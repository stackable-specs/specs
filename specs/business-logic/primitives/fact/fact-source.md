# Fact Source

Declares where a fact comes from and how trustworthy or fresh it is. Fact sources make data provenance a first-class part of the rule definition — so rules can behave differently based on data quality, recency, or verification status.

## Shape

```yaml
primitive: fact_source
name: <string>
trust_level: high | medium | low | unverified
freshness_limit: <duration>
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Identifier for this source (e.g. `verified_customer_address`, `credit_bureau_score`) |
| `trust_level` | enum | yes | How much the rule should rely on this source: `high`, `medium`, `low`, or `unverified` |
| `freshness_limit` | duration | no | Maximum age of data from this source before it is considered stale (e.g. `30d`, `1y`) |

## Use when

- A rule's outcome should differ based on whether data came from a verified vs. self-reported source
- Stale data should trigger a review or fallback rather than a confident decision
- Compliance or audit requirements demand that data provenance be recorded
- Multiple sources provide the same fact and precedence must be defined

## Composes with

- [fact](./fact.md) — a fact declares its source using this primitive
- [missing-fact](./missing-fact.md) — a stale or untrusted source can be treated as effectively missing
- [audit-requirement](../governance/audit-requirement.md) — source identity is often a required audit capture
- [enum-decision](../decision/enum-decision.md) — trust level can feed directly into a review/approve/reject decision

## Example

```yaml
primitive: fact_source
name: verified_customer_address
trust_level: high
freshness_limit: 365d
```

```yaml
primitive: fact_source
name: self_reported_annual_revenue
trust_level: unverified
freshness_limit: 90d
```
