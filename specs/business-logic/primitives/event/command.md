# Command

A requested action issued by an actor. Commands are distinct from events — a command requests something to happen, an event records that it did. A command can be rejected or deferred; the outcome is not guaranteed at the time of issuance.

## Shape

```yaml
primitive: command
name: SubmitInvoice
actor: user
target: invoice
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Pascal-case name of the command, phrased as an imperative verb + noun |
| `actor` | entity reference | yes | The entity issuing the command (user, service, scheduler) |
| `target` | entity reference | yes | The entity the command acts upon |
| `payload` | map | no | Data supplied with the command that shapes how it executes |
| `preconditions` | list of condition references | no | Conditions that must hold for the command to be accepted |

## Use when

- A user or upstream service is requesting a state-changing operation (approve, reject, submit, cancel, release)
- You need to distinguish the intent of an action from its result — the command is the intent, the event is the result
- A rule must validate or authorize a request before it takes effect
- You are modeling a workflow step that an actor deliberately initiates rather than one that fires automatically

## Composes with

- [trigger](./trigger.md) — a command can serve as the trigger source for a rule
- [event](./event.md) — a successfully executed command typically produces a corresponding event
- [entity](../entity/entity.md) — both actor and target are declared entities
- [exception](../exception/exception.md) — exceptions can block or redirect a command based on actor or target state

## Example

```yaml
primitive: command
name: SubmitInvoice
actor: user
target: invoice
payload:
  invoice_id: invoice.id
  submitted_by: user.id
preconditions:
  - invoice.status = Draft
  - invoice.line_items is not empty
```

```yaml
primitive: command
name: ApproveCreditLimit
actor: credit_analyst
target: account
payload:
  account_id: account.id
  new_limit: requested_limit
preconditions:
  - account.review_status = PendingApproval
  - credit_analyst.role in [CreditManager, VPFinance]
```
