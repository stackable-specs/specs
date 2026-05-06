# Gate

A hold point that blocks downstream progress until one or more conditions are satisfied. A gate makes the waiting explicit — it names what it is blocking, declares the condition that must be met to open, and can carry a timeout policy for cases where the condition is never satisfied.

## Shape

```yaml
primitive: gate
name: manual_review_gate
open_when:
  - review.status = Approved
blocks:
  - payout_release
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Unique identifier for this gate; referenced by the steps it blocks |
| `open_when` | list of conditions | yes | All conditions must be satisfied before the gate opens |
| `blocks` | list of step refs | yes | Steps or pipelines that cannot begin until this gate opens |
| `timeout` | duration | no | How long to wait before triggering the `on_timeout` handler |
| `on_timeout` | list of effects | no | Effects produced if the gate has not opened before `timeout` elapses |
| `notify_on_open` | list of actor refs | no | Actors or channels to notify when the gate opens |

## Use when

- A downstream step must not execute until an external party (human or system) has performed an action
- You are modelling a required approval before a financial disbursement, contract execution, or privileged state change
- The process has compliance requirements that mandate a waiting period or a sign-off before continuing
- You need a recoverable pause point — the gate stores its state so the pipeline can resume without replaying earlier steps
- You want to enforce separation of duties: the actor who initiates a process cannot also be the one to open the gate

## Composes with

- [pipeline](./pipeline.md) — gates are embedded as steps in a pipeline; the pipeline is paused at the gate and resumes when it opens
- [join](./join.md) — a gate can wait for multiple parallel approvals, similar to a join but driven by external state rather than internal step completion
- [rule](./rule.md) — `open_when` conditions are expressed using the same predicate syntax as rule conditions
- [governance](../governance/approval.md) — approval workflows use gates to enforce the required sign-off before a privileged action
- [event](../event/event.md) — the gate can emit an event when it opens, triggering downstream systems
- [actor](../entity/actor.md) — `notify_on_open` references actors who should be informed when the gate opens

## Example

```yaml
primitive: gate
name: manual_review_gate
open_when:
  - review.status = Approved
blocks:
  - payout_release
timeout: 72h
on_timeout:
  - escalate_to: compliance_manager
  - set_state: PayoutOnHold
```

```yaml
primitive: gate
name: contract_countersignature_gate
open_when:
  - contract.vendor_signature_received = true
  - contract.legal_review_passed = true
blocks:
  - procurement_order_creation
notify_on_open:
  - procurement_team
  - finance_controller
timeout: 14d
on_timeout:
  - emit_event: ContractExpired
  - archive_record: contract
```
