# Release

The conditions under which a named hold is automatically lifted from a business entity. Release pairs with hold to complete the hold lifecycle — it declares what must become true before the blocked operations are restored.

## Shape

```yaml
primitive: release
hold_type: CreditHold
release_when:
  - account.open_balance = 0
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `hold_type` | string | yes | The hold this release rule applies to — must match a declared `hold.hold_type` |
| `release_when` | list of predicates | yes | All conditions that must be true for the hold to be automatically lifted |

## Use when

- A hold should be lifted automatically when a measurable business condition is satisfied
- You need to decouple the logic for placing a hold from the logic for removing it
- Operations staff or automated jobs should be able to evaluate whether a hold qualifies for release without manual review
- The release criteria need to be auditable and traceable back to a specific business rule
- Multiple independent conditions must all be satisfied before the hold is cleared

## Composes with

- [hold](./hold.md) — release is the counterpart to hold; every hold should have at least one release rule
- [condition](../condition/condition.md) — release predicates are evaluated as conditions on the current state of the entity
- [fact](../fact/fact.md) — field values referenced in `release_when` predicates are sourced from facts
- [event](../event/event.md) — an event (e.g., PaymentPosted) often triggers a re-evaluation of release conditions
- [invariant](./invariant.md) — an invariant may assert that a hold must remain active while certain facts are true

## Example

```yaml
primitive: release
hold_type: CreditHold
release_when:
  - account.open_balance = 0
```

```yaml
primitive: release
hold_type: ComplianceHold
release_when:
  - employee.compliance_training_completed = true
  - employee.background_check_status = Cleared
```
