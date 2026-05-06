# Retention Requirement

Defines how long a record must be retained, and whether deletion or archival applies after that period. Retention requirements encode data lifecycle policy directly alongside the rules that produce records, keeping storage governance discoverable and enforceable at the rule level rather than buried in infrastructure configuration.

## Shape

```yaml
primitive: retention_requirement
record_type: DecisionRecord
retain_for: 7 years
basis: legal_requirement
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `record_type` | string | yes | The category of record this retention rule applies to (e.g., `DecisionRecord`, `AuditLog`, `ConsentRecord`) |
| `retain_for` | duration | yes | Minimum retention period expressed as a number and unit (e.g., `7 years`, `90 days`, `5 years`) |
| `basis` | enum | yes | The reason the retention period is required. One of: `legal_requirement`, `contractual_obligation`, `regulatory_mandate`, `business_policy`, `product_decision` |
| `after_period` | enum | no | Action to take when the retention period expires. One of: `delete`, `archive`, `anonymize`, `review`. Defaults to `review` |
| `policy_reference` | reference | no | Reference to the `policy-reference` primitive that cites the governing regulation or contract |
| `applies_to_regions` | string list | no | List of jurisdictions where this retention period applies (e.g., `EU`, `US`, `CA`) |

## Use when

- A rule produces records (decisions, audit logs, consent records) that are subject to regulatory or contractual retention mandates
- You need to enforce that records are not deleted prematurely (e.g., before a legal hold expires)
- Different record types have different lifecycle policies and you need to make each one explicit
- A privacy or compliance review requires proving that data is not kept longer than necessary
- You want to trigger automatic archival or deletion workflows at the end of the retention window

## Composes with

- [audit-requirement](./audit-requirement.md) — audit records are the most common target of a retention requirement; compose them so the audit requirement references the correct retention policy
- [privacy-constraint](./privacy-constraint.md) — retention requirements define how long data can be held; privacy constraints define what it can be used for during that window
- [policy-reference](./policy-reference.md) — the `basis` of a retention requirement almost always traces to a specific regulation or contract clause
- [../composition/rule](../composition/rule.md) — a rule that produces long-lived records should declare the applicable retention requirement at design time

## Example

```yaml
primitive: retention_requirement
record_type: DecisionRecord
retain_for: 7 years
basis: legal_requirement
after_period: archive
policy_reference: GDPR-Art-5-1e
applies_to_regions:
  - EU
```

```yaml
primitive: retention_requirement
record_type: ConsentRecord
retain_for: 5 years
basis: regulatory_mandate
after_period: delete
policy_reference: CCPA-1798-105
applies_to_regions:
  - US
  - CA
```
