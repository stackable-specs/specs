# Conflict Resolver

Detects contradictory outcomes produced by multiple rules and resolves them according to a defined strategy. Where [precedence](./precedence.md) ranks overlapping outcomes, a conflict resolver handles structurally incompatible ones — for example, two rules that simultaneously assert `refund_approved = true` and `refund_approved = false`, or two rules that create duplicate obligations for the same subject.

## Shape

```yaml
primitive: conflict_resolver
detect:
  - contradictory_outcomes
  - duplicate_obligations
resolve_by:
  - highest_priority
  - most_specific
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `detect` | list of enums | yes | Categories of conflict to watch for |
| `resolve_by` | list of strategies | yes | Ordered list of resolution strategies applied in sequence until the conflict is resolved |
| `scope` | list of field paths | no | Limit detection to specific output fields; if omitted, all outputs are inspected |
| `on_unresolvable` | list of effects | no | Effects produced when no strategy succeeds — typically an escalation or a hold |

### `detect` values

| Value | Meaning |
|-------|---------|
| `contradictory_outcomes` | Two rules assert opposite boolean values for the same field |
| `duplicate_obligations` | Two rules create the same obligation type for the same subject |
| `overlapping_amounts` | Two rules assign different numeric values to the same field |
| `mutually_exclusive_states` | Two rules attempt to set the same entity to states that cannot coexist |

### `resolve_by` strategies

| Strategy | Meaning |
|----------|---------|
| `highest_priority` | Defer to the [precedence](./precedence.md) declaration for the conflicting rules |
| `most_specific` | Keep the outcome from the rule with the most conditions satisfied |
| `most_restrictive` | Choose the stricter of the two conflicting values |
| `most_permissive` | Choose the looser of the two conflicting values |
| `escalate` | Surface the conflict to a human reviewer instead of resolving automatically |

## Use when

- Multiple independently authored rules operate on the same entity and can fire simultaneously
- A rule produces a boolean flag (approved, eligible, blocked) and another rule produces the opposite value for the same flag
- Obligations (tasks, notifications, charges) must be deduplicated to prevent double-billing or duplicate work orders
- You are operating in a regulatory context where an unresolved conflict must be escalated rather than silently resolved
- You are introducing a new rule alongside existing ones and want to catch conflicts at evaluation time before they reach downstream systems

## Composes with

- [rule](./rule.md) — conflict resolution operates over the output set produced by multiple rules
- [rule-set](./rule-set.md) — a rule set with `collect_all` mode is the most common source of conflicting outputs that need resolution
- [precedence](./precedence.md) — the `highest_priority` resolution strategy delegates to a precedence declaration
- [governance](../governance/approval.md) — when `resolve_by` includes `escalate`, the escalation path is typically an approval workflow
- [exception](../exception/exception.md) — an unresolvable conflict can be surfaced as a typed exception for upstream handling

## Example

```yaml
primitive: conflict_resolver
detect:
  - contradictory_outcomes
  - duplicate_obligations
resolve_by:
  - highest_priority
  - most_specific
scope:
  - order.discount_rate
  - order.approval_required
on_unresolvable:
  - escalate_to: pricing_operations
  - set_state: OrderOnHold
```

```yaml
primitive: conflict_resolver
detect:
  - mutually_exclusive_states
  - overlapping_amounts
resolve_by:
  - most_restrictive
  - escalate
scope:
  - claim.payout_amount
  - claim.status
on_unresolvable:
  - emit_event: ClaimConflictDetected
  - create_obligation: ManualAdjudicationTask
```
