# Policy Reference

Links a rule to its authoritative source — a contract clause, regulation, internal policy, or product decision. Policy references make rules traceable to the business decisions that created them, enabling compliance audits, dispute resolution, and impact analysis when source documents change.

## Shape

```yaml
primitive: policy_reference
source_type: contract
source_id: MSA-123
clause: "4.2"
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `source_type` | enum | yes | Category of the authoritative source. One of: `contract`, `regulation`, `policy`, `requirement`, `product_decision` |
| `source_id` | string | yes | Identifier of the source document (e.g., contract number, regulation code, Jira ticket) |
| `clause` | string | no | Specific section, clause, or article within the source document |
| `description` | string | no | Human-readable summary of what the referenced clause requires |
| `url` | string | no | Link to the canonical source document or section |

## Use when

- A rule exists because of a specific contractual obligation, regulation, or executive decision and you need to prove it
- A compliance audit requires mapping each decision rule to its governing document
- You want to detect which rules are affected when a contract or regulation changes
- A rule may be challenged and you need a clear chain of authority back to the source
- You are tracking which product decisions have been encoded as executable rules

## Composes with

- [rule-version](./rule-version.md) — policy references should be re-evaluated whenever a rule version changes, in case the underlying clause was updated
- [audit-requirement](./audit-requirement.md) — audit records should include the policy reference so that log entries are self-contained for compliance review
- [approval-requirement](./approval-requirement.md) — approvals are often mandated by the same policy document that governs the rule
- [../composition/rule](../composition/rule.md) — attach a policy reference to a rule to make the rule's legal basis explicit

## Example

```yaml
primitive: policy_reference
source_type: contract
source_id: MSA-123
clause: "4.2"
description: "Refunds must be processed within 5 business days of approval."
url: https://contracts.internal/msa-123#clause-4-2
```

```yaml
primitive: policy_reference
source_type: regulation
source_id: GDPR-Art-17
clause: "Article 17(1)"
description: "Right to erasure — personal data must be deleted upon verified request."
url: https://gdpr-info.eu/art-17-gdpr/
```
