# Block Action

Prevents a named action from proceeding, attaching a machine-readable reason code that callers can inspect. The block is declarative — any system attempting to execute the named action consults the rule engine first, and if a `block_action` effect is active, the action is rejected before it takes effect. This is the enforcement primitive for access-control and compliance rules.

## Shape

```yaml
primitive: block_action
action: ShipOrder
reason_code: CreditHold
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `action` | string | yes | Pascal-case name of the action being blocked |
| `reason_code` | string | yes | Machine-readable code describing why the action is blocked; used by callers to display messages or route exceptions |
| `message` | string | no | Human-readable explanation to surface in UIs or error responses |
| `expires_at` | timestamp or expression | no | If set, the block automatically lifts after this time; otherwise the block persists until a matching `allow_action` effect fires |

## Use when

- A business policy must prevent an action from executing when the entity is in an invalid or risky state (e.g., block shipment when the account is on credit hold)
- Compliance or regulatory rules require hard stops on certain operations (e.g., block refund approval above a threshold without a supervisor override)
- You need a caller-visible reason code so that UIs, APIs, and downstream services can respond appropriately without hardcoding business logic
- A default-allow policy needs selective guards inserted for specific conditions
- The block is temporary and tied to a condition that will eventually resolve (e.g., block order placement while a fraud review is open)

## Composes with

- [allow-action](./allow-action.md) — a matching `allow_action` on the same action name can override or lift this block in a default-deny arrangement
- [condition](../condition/predicate.md) — the rule's condition determines when the block fires; the block itself carries only the enforcement payload
- [transition](../state/transition.md) — transitions with preconditions that fail are often surfaced as `block_action` effects to give callers a structured response
- [notify](./notify.md) — pair with a `notify` effect to alert an operator or customer when an action is blocked
- [create-obligation](./create-obligation.md) — when unblocking requires a human task, create an obligation alongside the block

## Example

```yaml
primitive: block_action
action: ShipOrder
reason_code: CreditHold
message: "Shipment is blocked because the account has an outstanding credit hold."
```

```yaml
primitive: block_action
action: ApproveRefund
reason_code: ThresholdExceeded
message: "Refunds above $500 require supervisor approval."
expires_at: null
```
