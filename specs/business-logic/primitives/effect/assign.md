# Assign

Routes a work item to a person, role, or queue so that the right party takes ownership. The assignment effect records who is responsible without specifying how the work is executed — the assignee can be a named individual, a named role, or an abstract queue that a scheduling system drains. Use it whenever a rule must move a work item from "unassigned" to "owned."

## Shape

```yaml
primitive: assign
work_item: ticket
assignee: EnterpriseSupportQueue
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `work_item` | entity reference or field path | yes | The entity being routed (e.g., `ticket`, `order`, `task`); must reference a declared entity |
| `assignee` | string or field path | yes | The target person, role, or queue receiving the work item; may be a literal queue name or a field path resolving to an agent ID |
| `priority` | enum or integer | no | Optional priority tier to attach to the assignment (e.g., `high`, `normal`, `low`, or a numeric rank) |
| `reason` | string | no | Human-readable explanation of why this routing decision was made, for audit and reporting |
| `replace_existing` | boolean | no | If `true`, overwrites any existing assignee; if `false` (default), the assignment is skipped if the work item already has an owner |

## Use when

- A rule must route an inbound request, ticket, or task to the appropriate team or individual based on entity attributes (e.g., route enterprise accounts to the enterprise support queue)
- A work item changes state and ownership must transfer simultaneously (e.g., re-assign a dispute to the fraud review team when it is escalated)
- Load-balancing or skill-based routing decisions need to be captured as explicit rule effects rather than embedded in queue management code
- A newly created record (from `create_record`) needs an owner set in the same rule execution
- Escalation logic must shift ownership from a tier-1 queue to a tier-2 specialist

## Composes with

- [create-record](./create-record.md) — `assign` typically follows a `create_record` to give the new work item an owner immediately
- [create-obligation](./create-obligation.md) — an obligation on the assignee is the natural companion: "AssignedTo X, must respond by Y"
- [notify](./notify.md) — notify the assignee that a new item has arrived in their queue
- [entity](../entity/entity.md) — the `work_item` must be a declared entity so field paths resolve correctly
- [decision](../decision/selection.md) — a `selection` decision can compute which queue or agent to assign to before the `assign` effect fires

## Example

```yaml
primitive: assign
work_item: ticket
assignee: EnterpriseSupportQueue
priority: high
reason: Account MRR above enterprise threshold
```

```yaml
primitive: assign
work_item: dispute
assignee: FraudReviewTeam
priority: urgent
replace_existing: true
reason: Dispute escalated after initial review found suspicious pattern
```
