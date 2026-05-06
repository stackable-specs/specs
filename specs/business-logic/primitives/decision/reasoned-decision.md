# Reasoned Decision

A decision that carries its result alongside the reason codes and evidence that produced it. Reasoned decisions make the "why" a first-class output — suitable for audit logs, customer-facing explanations, dispute resolution, and downstream rules that branch on the reason rather than the result alone.

## Shape

```yaml
primitive: reasoned_decision
decision: refund_eligibility
result: Ineligible
reason_codes:
  - FinalSale
  - OutsideReturnWindow
evidence:
  - order.delivered_at
  - item.final_sale
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `decision` | string | yes | Snake-case name of the decision being made; links this record to the boolean or enum decision it wraps |
| `result` | string or boolean | yes | The resolved outcome value — a boolean (`true`/`false`) or a named enum value |
| `reason_codes` | string list | yes | One or more reason codes that explain why this result was reached; codes should be defined in the reason-code catalog |
| `evidence` | field path list | no | The specific field values observed at decision time that activated the reason codes; used for traceability and dispute resolution |

## Use when

- A decision outcome must be explainable to a customer, agent, regulator, or auditor
- Downstream rules or workflow steps need to branch on the *reason* for a decision, not just its result (e.g. different denial letters for different reason codes)
- Multiple conditions could produce the same result but require distinct follow-up actions or disclosures
- The decision is subject to appeal or dispute and the evidence must be preserved at point-in-time
- Compliance requirements mandate a documented rationale for approvals, denials, or eligibility rulings

## Composes with

- [boolean-decision](./boolean-decision.md) — a reasoned decision wraps a boolean decision to attach explanation to its true/false result
- [enum-decision](./enum-decision.md) — a reasoned decision wraps an enum decision to explain which value was selected and why
- [reason-code](../explanation/reason-code.md) — reason codes listed in `reason_codes` must be defined in the explanation catalog with their human-readable descriptions
- [fact](../fact/fact.md) — fields listed in `evidence` are sourced from facts; the fact primitive records the value observed at evaluation time
- [classification](./classification.md) — pair with classification when the result is a tier or category and the explanation must identify which threshold triggered the assignment

## Example

```yaml
primitive: reasoned_decision
decision: refund_eligibility
result: Ineligible
reason_codes:
  - FinalSale
  - OutsideReturnWindow
evidence:
  - order.delivered_at
  - item.final_sale
```

```yaml
primitive: reasoned_decision
decision: loan_application_disposition
result: Denied
reason_codes:
  - InsufficientCreditScore
  - DebtToIncomeExceedsLimit
evidence:
  - applicant.credit_score
  - applicant.monthly_debt
  - applicant.gross_monthly_income
```
