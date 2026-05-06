# Pipeline

A sequence of named steps where each step's output is available to the next. Pipelines encode multi-stage processes where order is meaningful — a later step can depend on state produced by an earlier one, and a failure in any step halts the sequence unless the step is marked optional.

## Shape

```yaml
primitive: pipeline
steps:
  - validate_order
  - reserve_inventory
  - authorize_payment
  - create_shipment
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | no | Optional identifier so this pipeline can be referenced by other primitives |
| `steps` | list | yes | Ordered list of step references; evaluated top-to-bottom |
| `steps[].name` | string | yes (if object form) | Name of the step (rule, rule set, or sub-pipeline) |
| `steps[].optional` | boolean | no | When `true`, a failure in this step does not halt the pipeline (default `false`) |
| `steps[].on_failure` | string | no | Alternative step to execute if this step fails |
| `on_complete` | list of effects | no | Effects produced when all steps finish successfully |
| `on_failure` | list of effects | no | Effects produced when the pipeline halts due to a step failure |

## Use when

- A business process has a defined start-to-finish sequence where earlier stages must succeed before later ones begin
- You need data produced in step N to be visible to step N+1 (e.g. a payment authorization token used by the shipment creation step)
- Partial execution needs to be detectable and recoverable — a pipeline makes the last-completed step explicit
- You are modelling an order fulfillment flow, a loan origination process, a customer onboarding sequence, or any other staged workflow
- You need to attach compensating effects to specific failure points rather than handling all failures uniformly

## Composes with

- [rule](./rule.md) — individual pipeline steps are typically rules or rule sets
- [rule-set](./rule-set.md) — a step can delegate to a named rule set for multi-condition evaluation
- [branch](./branch.md) — a step can be a branch that forks execution before rejoining the pipeline
- [gate](./gate.md) — a step can be a gate that holds the pipeline until an external condition is satisfied
- [join](./join.md) — parallel branches launched within a pipeline are collected by a join step before the pipeline continues
- [event](../event/event.md) — pipeline steps can emit events consumed by other systems

## Example

```yaml
primitive: pipeline
name: order_fulfillment
steps:
  - validate_order
  - reserve_inventory
  - authorize_payment
  - create_shipment
on_failure:
  - notify: operations_team
  - set_state: OrderFailed
```

```yaml
primitive: pipeline
name: loan_origination
steps:
  - collect_application
  - run_credit_check
  - validate_income_documents
  - name: manual_underwriter_review
    optional: false
    on_failure: escalate_to_senior_underwriter
  - generate_loan_offer
  - capture_borrower_acceptance
on_complete:
  - emit_event: LoanApproved
  - create_obligation: DisburseProceeds
```
