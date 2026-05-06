# Terminal State

A state value from which no further transitions are permitted. Terminal states represent the final resting points of an entity's lifecycle — they signal that processing is complete and the record is closed for modification.

## Shape

```yaml
primitive: terminal_state
entity: invoice
states:
  - Paid
  - Cancelled
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `entity` | string | yes | The business entity whose lifecycle ends at these states |
| `states` | list of strings | yes | The state values that are terminal — no outgoing transitions may originate from these values |

## Use when

- An entity has one or more end-of-life states where no further business events should be processed
- You need to prevent reactivation or modification of a closed record
- Downstream systems need to know when an entity's lifecycle is definitively complete
- Archival, retention, or audit policies apply only to records that have reached a terminal point
- An invariant must hold permanently once a terminal state is entered

## Composes with

- [state](./state.md) — terminal state values must be members of the entity's declared state value set
- [transition](./transition.md) — transitions with a `to` value listed here have no further outgoing transitions defined
- [invariant](./invariant.md) — invariants enforced in terminal states can never be repaired by a future transition
- [hold](./hold.md) — a hold may automatically release or become irrelevant when a terminal state is entered
- [event](../event/event.md) — events that arrive after a terminal state is reached should be rejected or routed to a dead-letter handler

## Example

```yaml
primitive: terminal_state
entity: invoice
states:
  - Paid
  - Cancelled
```

```yaml
primitive: terminal_state
entity: support_ticket
states:
  - Resolved
  - Closed
  - Withdrawn
```
