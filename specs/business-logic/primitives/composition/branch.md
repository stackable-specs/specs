# Branch

A conditional fork that routes execution to exactly one of two named paths based on a single condition. A branch does not evaluate multiple conditions — it makes a binary decision and hands off to the chosen path. If more than two paths are needed, branches can be nested or replaced with a [decision-table](./decision-table.md).

## Shape

```yaml
primitive: branch
if: payment.status = Failed
then: dunning_flow
else: fulfillment_flow
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `if` | condition expression | yes | The condition evaluated to determine which path to take |
| `then` | step ref | yes | The step, pipeline, or rule set to invoke when the condition is true |
| `else` | step ref | no | The step, pipeline, or rule set to invoke when the condition is false; if omitted, the false path is a no-op |
| `name` | string | no | Optional identifier so this branch can be referenced by other primitives |

## Use when

- Execution must split into two distinct paths and only one should run for a given input
- The condition is a single boolean test rather than a multi-dimensional matrix (use [decision-table](./decision-table.md) for the latter)
- You are routing between a happy path and an exception path (success vs. failure, eligible vs. ineligible, approved vs. declined)
- The two paths eventually rejoin — the branch is contained within a [pipeline](./pipeline.md) and a [join](./join.md) or subsequent step merges them back
- You need to make the routing logic explicit and separately auditable from the steps it routes to

## Composes with

- [pipeline](./pipeline.md) — a branch is commonly a step inside a pipeline that forks execution before the pipeline continues
- [rule-set](./rule-set.md) — `then` and `else` targets are often named rule sets or sub-pipelines
- [condition](../condition/condition.md) — the `if` field references a condition expression or named condition
- [gate](./gate.md) — the `then` or `else` path may start with a gate that holds execution until an approval is received
- [join](./join.md) — when the two branches run in parallel and must be synchronized before continuing

## Example

```yaml
primitive: branch
if: payment.status = Failed
then: dunning_flow
else: fulfillment_flow
```

```yaml
primitive: branch
name: refund_path_selector
if: order.days_since_delivery <= 30
then: self_service_refund_pipeline
else:
  name: manual_refund_review_pipeline
```

```yaml
primitive: branch
name: kyc_routing
if: customer.risk_score > 75
then: enhanced_due_diligence_pipeline
else: standard_onboarding_pipeline
```
