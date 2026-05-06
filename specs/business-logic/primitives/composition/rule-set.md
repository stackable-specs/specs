# Rule Set

A named group of rules evaluated together under a defined evaluation mode. A rule set is the primary way to express a policy that has multiple cases — rather than writing one monolithic rule, you compose smaller focused rules and declare how their results combine.

## Shape

```yaml
primitive: rule_set
rules:
  - refund_eligibility
  - refund_calculation
  - refund_approval_requirement
evaluation_mode: collect_all
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `rules` | list of rule refs | yes | Ordered list of rule names to evaluate |
| `evaluation_mode` | enum | yes | How results from individual rules are combined |
| `name` | string | no | Optional identifier so this set can be referenced by other primitives |

### `evaluation_mode` values

| Mode | Meaning |
|------|---------|
| `first_match` | Stop after the first rule whose conditions pass; use its effects |
| `collect_all` | Evaluate every rule; accumulate all effects that fire |
| `all_must_pass` | Treat the set as failed if any rule's conditions do not hold |
| `any_may_pass` | Succeed if at least one rule's conditions hold |
| `priority_order` | Evaluate in priority order; stop when a rule fires (like `first_match` but driven by explicit priority, not list position) |

## Use when

- A business policy is naturally expressed as several independent sub-rules that all apply to the same subject
- You need a single evaluation context whose aggregate outcome determines downstream routing (e.g. a [branch](./branch.md) or [gate](./gate.md))
- Rules are maintained by different teams but must be evaluated together (pricing overrides, discount stacking, approval chains)
- You want to isolate the evaluation strategy from the rule definitions themselves so it can be changed without rewriting the rules
- You need to guarantee that all rules in a compliance checklist pass before allowing an action

## Composes with

- [rule](./rule.md) — the individual units collected into this set
- [precedence](./precedence.md) — when `evaluation_mode: priority_order` is used, precedence declarations drive the ordering
- [conflict-resolver](./conflict-resolver.md) — resolves contradictions when `collect_all` produces incompatible effects
- [decision-table](./decision-table.md) — an alternative form when the rule set is driven by a matrix of input combinations
- [pipeline](./pipeline.md) — a rule set can be one step in a larger pipeline

## Example

```yaml
primitive: rule_set
name: refund_policy
rules:
  - refund_eligibility
  - refund_calculation
  - refund_approval_requirement
evaluation_mode: collect_all
```

```yaml
primitive: rule_set
name: discount_stack
rules:
  - loyalty_discount
  - volume_discount
  - promotional_discount
  - employee_discount
evaluation_mode: first_match
```
