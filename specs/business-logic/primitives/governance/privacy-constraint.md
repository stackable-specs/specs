# Privacy Constraint

Restricts how a data field may be used, to which purposes it can be applied, and under what consent conditions access is permitted. Privacy constraints encode data-use policy at the field level so that rules consuming sensitive data remain compliant regardless of where they are composed or reused.

## Shape

```yaml
primitive: privacy_constraint
data_field: customer.email
allowed_purposes:
  - transactional_notice
  - marketing_if_consented
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `data_field` | field path | yes | Dot-path to the sensitive field this constraint governs |
| `allowed_purposes` | string list | yes | Exhaustive list of purposes for which this field may be used |
| `requires_consent` | string list | no | Subset of `allowed_purposes` that require explicit user consent before use |
| `prohibited_in` | string list | no | Named contexts or rule families where use of this field is explicitly forbidden |
| `sensitivity_class` | enum | no | Classification of the field's sensitivity. One of: `public`, `internal`, `confidential`, `restricted`, `pii`, `sensitive_pii` |
| `policy_reference` | reference | no | Reference to the `policy-reference` primitive that mandates this constraint |
| `mask_in_logs` | boolean | no | If true, this field must be masked or tokenized in all audit and execution logs |

## Use when

- A rule reads a field that contains personal, financial, or otherwise sensitive data
- You need to enforce purpose limitation — preventing a field collected for one purpose from being used for another
- Consent-gated data use must be declared explicitly so the runtime can enforce it before evaluation
- A privacy impact assessment requires documentation of which fields are used by which rules and for which purposes
- You want to prevent PII from appearing in plain-text audit logs or metrics

## Composes with

- [retention-requirement](./retention-requirement.md) — privacy constraints govern what data can be used for; retention requirements govern how long it can be kept; both apply to the same sensitive field
- [audit-requirement](./audit-requirement.md) — `mask_in_logs` pairs with the audit requirement to ensure compliant log hygiene
- [policy-reference](./policy-reference.md) — the allowed purposes and consent requirements originate from a regulation or policy document
- [../fact/fact](../fact/fact.md) — a privacy constraint is attached to the fact that sources the sensitive field, propagating the restriction to every rule that reads it

## Example

```yaml
primitive: privacy_constraint
data_field: customer.email
allowed_purposes:
  - transactional_notice
  - marketing_if_consented
requires_consent:
  - marketing_if_consented
sensitivity_class: pii
mask_in_logs: true
policy_reference: GDPR-Art-6
```

```yaml
primitive: privacy_constraint
data_field: customer.date_of_birth
allowed_purposes:
  - age_verification
  - eligibility_check
prohibited_in:
  - analytics_aggregation
  - marketing_segmentation
sensitivity_class: sensitive_pii
mask_in_logs: true
policy_reference: COPPA-Sec-312
```
