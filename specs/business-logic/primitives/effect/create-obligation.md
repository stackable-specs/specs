# Create Obligation

Creates a required task or duty with an explicit owner and a due date. An obligation is stronger than a task — it is a tracked commitment that the system expects to be fulfilled, and its non-fulfillment is itself a business event that can trigger escalation rules. Use it when a rule outcome must produce a time-bound accountability, not merely a suggestion.

## Shape

```yaml
primitive: create_obligation
type: RespondToTicket
owner: SupportTeam
due_at: ticket.created_at + 4 business_hours
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | string | yes | Pascal-case name of the obligation class (e.g., `RespondToTicket`, `ReviewDispute`, `ConductAudit`) |
| `owner` | string or field path | yes | The person, role, or team responsible for fulfilling the obligation |
| `due_at` | timestamp or expression | yes | Deadline by which the obligation must be fulfilled; supports business-hours arithmetic |
| `subject` | entity reference | no | The entity the obligation relates to (e.g., the ticket, invoice, or contract at hand) |
| `escalates_to` | string | no | If the obligation is not fulfilled by `due_at`, ownership transfers to this role or queue |
| `evidence_field` | field path | no | Field on the subject entity that, when set, is treated as proof of fulfillment |

## Use when

- A service-level agreement (SLA) or policy requires that a named party act within a specific time window (e.g., support must respond within 4 business hours)
- A compliance or regulatory rule mandates a documented, trackable duty (e.g., a KYC review must be completed within 5 business days of account opening)
- A rule must create accountability alongside a work item — the obligation tracks whether the work was done on time, not just whether it was assigned
- Escalation logic depends on whether a prior obligation was fulfilled or breached
- You need a structured, queryable record of outstanding duties for reporting and SLA dashboards

## Composes with

- [assign](./assign.md) — the obligation owner often matches the work item assignee; create both effects in the same rule
- [notify](./notify.md) — notify the obligation owner at creation time, and again as the deadline approaches
- [create-record](./create-record.md) — an obligation may be modeled as a specialized record type; `create_obligation` is the semantic primitive, `create_record` is the persistence mechanism
- [event](../event/event.md) — a `ObligationBreached` event fires when `due_at` passes without fulfillment, triggering escalation rules
- [block-action](./block-action.md) — block downstream actions until an outstanding obligation is fulfilled (e.g., block invoice payment release until a compliance review obligation is closed)

## Example

```yaml
primitive: create_obligation
type: RespondToTicket
owner: SupportTeam
due_at: ticket.created_at + 4 business_hours
subject: ticket
escalates_to: SupportManager
evidence_field: ticket.first_response_at
```

```yaml
primitive: create_obligation
type: ReviewDispute
owner: FraudAnalystQueue
due_at: dispute.opened_at + 3 business_days
subject: dispute
escalates_to: FraudDirector
evidence_field: dispute.reviewed_at
```
