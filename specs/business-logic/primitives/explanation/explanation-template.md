# Explanation Template

A human-readable message template for communicating a decision outcome to a specific audience. Explanation templates decouple the machine-readable reason codes produced by rules from the audience-appropriate language used to communicate them — so the same decision can be explained plainly to a customer, precisely to a compliance analyst, and verbosely to a regulator, without changing the rule.

## Shape

```yaml
primitive: explanation_template
audience: customer
template: "Your refund was not approved because {reason_codes}."
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `audience` | enum | yes | The intended recipient of this explanation. One of: `customer`, `agent`, `analyst`, `regulator` |
| `template` | string | yes | Message template with `{placeholder}` slots for dynamic values. Standard placeholders: `{reason_codes}`, `{outcome}`, `{evidence}`, `{rule_id}`, `{decision_id}` |
| `outcome` | string | no | Scopes the template to a specific outcome (e.g., `denied`, `approved`, `flagged`). If omitted, the template applies to all outcomes for this audience |
| `locale` | string | no | BCP 47 locale code for the language and region of this template (e.g., `en-US`, `fr-FR`). Defaults to `en-US` |
| `format` | enum | no | Output format of the rendered message. One of: `plain_text`, `markdown`, `html`. Defaults to `plain_text` |

## Audience options

| Value | Characteristics |
|-------|----------------|
| `customer` | Plain language, empathetic tone, minimal jargon, actionable next steps |
| `agent` | Concise, includes reason codes and relevant evidence, oriented toward resolution |
| `analyst` | Verbose, includes rule IDs, version, all matched conditions, and evidence values |
| `regulator` | Formal, includes policy references, full evidence, and audit record identifiers |

## Use when

- A rule produces an outcome that must be communicated to more than one type of audience with different vocabulary and detail needs
- You want to avoid hard-coding customer-facing messages in application code and instead keep them as configurable data alongside the rule
- Localization is required and you need a locale-scoped template per audience per language
- A compliance team needs a regulator-ready explanation format that is consistent across all decisions of a given type
- You are building an agent or support workflow that generates explanations dynamically from reason codes

## Composes with

- [reason-code](./reason-code.md) — reason codes are the primary dynamic input interpolated into templates via the `{reason_codes}` placeholder
- [evidence](./evidence.md) — evidence values can be injected via the `{evidence}` placeholder to show exactly what data drove the decision
- [decision-trace](./decision-trace.md) — a decision trace provides the full context (rules, conditions, outcome) from which an explanation template can be rendered
- [../governance/audit-requirement](../governance/audit-requirement.md) — rendered explanations sent to customers or regulators should themselves be audited; the template identifier and rendered text belong in the audit record

## Example

```yaml
primitive: explanation_template
audience: customer
outcome: denied
template: "We were unable to process your refund for order {order_id}. {reason_codes}. If you believe this is an error, please contact support."
locale: en-US
format: plain_text
```

```yaml
primitive: explanation_template
audience: analyst
outcome: denied
template: "Decision: {outcome} | Rule: {rule_id} v{rule_version} | Reasons: {reason_codes} | Evidence: {evidence} | Trace ID: {decision_id}"
format: plain_text
```
