# Join

Waits for multiple parallel steps to complete before allowing execution to continue. A join is the complement to a [branch](./branch.md) that spawned parallel paths — it collects their results and merges them into a single execution context before handing off to the next step.

## Shape

```yaml
primitive: join
wait_for:
  - fraud_review_completed
  - payment_authorized
mode: all
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `wait_for` | list of step refs | yes | The parallel steps or signals whose completion this join is listening for |
| `mode` | enum | yes | Determines how many completions are required to unblock the join |
| `name` | string | no | Optional identifier so this join can be referenced by other primitives |
| `timeout` | duration | no | How long to wait before triggering the `on_timeout` handler |
| `on_timeout` | list of effects | no | Effects produced if the join has not resolved before `timeout` elapses |
| `on_complete` | list of effects | no | Effects produced immediately when the join resolves |

### `mode` values

| Mode | Meaning |
|------|---------|
| `all` | Every step in `wait_for` must complete before the join resolves |
| `any` | The join resolves as soon as the first step in `wait_for` completes |
| `first` | Equivalent to `any` but cancels all remaining steps once the first completes |

## Use when

- A process spawned parallel sub-processes (via a [branch](./branch.md) or explicit fork) and downstream logic requires results from more than one of them
- Multiple independent checks (fraud, credit, compliance) must each finish before a single gated action (disbursement, shipment) can proceed
- You are using `mode: any` to implement a race — whichever approval comes back first is sufficient to unblock the process
- You need a clean synchronization boundary that makes the parallel-to-sequential transition explicit and auditable
- You want timeout handling when one parallel path is slow or unresponsive without failing the entire process

## Composes with

- [branch](./branch.md) — branches fork execution; joins collect it back
- [pipeline](./pipeline.md) — a join is embedded as a step in a pipeline, sitting after the parallel steps and before the steps that depend on their results
- [gate](./gate.md) — a gate blocks on an external condition; a join blocks on the completion of internal steps — they are complementary hold points
- [event](../event/event.md) — the `wait_for` list can reference events emitted by external systems rather than internal steps
- [rule-set](./rule-set.md) — `on_complete` can invoke a rule set that acts on the merged results of all parallel steps

## Example

```yaml
primitive: join
name: pre_disbursement_sync
wait_for:
  - fraud_review_completed
  - payment_authorized
mode: all
timeout: 4h
on_timeout:
  - notify: risk_operations
  - set_state: DisbursementOnHold
```

```yaml
primitive: join
name: fastest_carrier_selection
wait_for:
  - fedex_rate_quote_received
  - ups_rate_quote_received
  - dhl_rate_quote_received
mode: first
on_complete:
  - assign: shipment.carrier = first_responder.carrier
  - assign: shipment.quoted_cost = first_responder.rate
```
