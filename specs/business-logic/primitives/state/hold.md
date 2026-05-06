# Hold

A named suspension placed on a business entity that blocks one or more operations until the hold is explicitly released. Holds make blocking conditions first-class: they are named, inspectable, and auditable rather than buried in conditional logic.

## Shape

```yaml
primitive: hold
entity: account
hold_type: CreditHold
blocks:
  - place_order
  - ship_order
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `entity` | string | yes | The business entity on which the hold is placed |
| `hold_type` | string | yes | PascalCase name that uniquely identifies this kind of hold |
| `blocks` | list of strings | yes | Operations that cannot proceed while this hold is active |

## Use when

- A business condition must prevent specific operations without necessarily changing the entity's primary state value
- Multiple independent reasons can block the same operation and each must be individually tracked and released
- Operations must fail fast with a named, auditable reason rather than a generic validation error
- Customer service or operations teams need to see which holds are active and why
- A hold must be placeable and releasable by different actors or systems operating independently

## Composes with

- [release](./release.md) — defines the conditions under which this hold is lifted
- [state](./state.md) — holds often coexist with a state machine; the entity's state may remain Active while a hold blocks certain operations
- [invariant](./invariant.md) — an invariant may assert that certain holds must be present when specific conditions are true
- [condition](../condition/condition.md) — conditions evaluate whether a hold is active before allowing downstream logic
- [event](../event/event.md) — an event is typically the trigger that places or removes a hold
- [actor](../entity/actor.md) — the actor responsible for placing the hold should be captured for audit purposes

## Example

```yaml
primitive: hold
entity: account
hold_type: CreditHold
blocks:
  - place_order
  - ship_order
```

```yaml
primitive: hold
entity: employee
hold_type: ComplianceHold
blocks:
  - approve_expense
  - submit_timesheet
  - process_payroll
```
