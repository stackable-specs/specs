# Exception

A named condition that overrides the base rule outcome. Exceptions are not errors — they are anticipated deviations with defined effects. Every exception is a first-class business concept that should be named, documented, and auditable, not buried in conditional logic.

## Shape

```yaml
primitive: exception
name: StrategicCustomerOverride
applies_when:
  - customer.tier = Strategic
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Pascal-case name that describes the business reason for the deviation |
| `applies_when` | list of conditions | yes | One or more conditions that must all hold for the exception to activate |
| `effect` | outcome map | no | The alternative outcome to apply when the exception is active; if omitted, the exception suppresses the base outcome entirely |
| `priority` | integer | no | Evaluation order when multiple exceptions could apply; higher value wins |
| `requires_approval` | boolean | no | If true, a human must approve activation of this exception before its effect is applied |
| `audit_reason` | string | no | A required human-readable justification captured when the exception fires |

## Use when

- A segment of customers, accounts, or contracts has a known, sanctioned deviation from standard policy
- A rule's base outcome would be incorrect for a well-understood class of cases and the deviation has a name in the business domain
- You need a traceable record of why a standard rule did not apply for a given entity instance
- Regulatory or contractual obligations create a defined carve-out from the default logic

## Composes with

- [override](./override.md) — an exception often delegates its effect to an override, which specifies the replacement outcome and authority chain
- [waiver](./waiver.md) — an exception can trigger a waiver when the deviation is specifically about forgiving a fee or obligation
- [fallback](./fallback.md) — if no exception matches and no rule matches, the fallback applies
- [condition](../condition/condition.md) — the `applies_when` clauses are standard conditions referencing entity fields

## Example

```yaml
primitive: exception
name: StrategicCustomerOverride
applies_when:
  - customer.tier = Strategic
effect:
  credit_hold: false
  collections_outreach: suppressed
priority: 10
audit_reason: "Strategic accounts are exempt from automated credit holds per commercial agreement."
```

```yaml
primitive: exception
name: PilotProgramPricingException
applies_when:
  - account.program = EarlyAdopterPilot
  - contract.signed_before = 2024-01-01
effect:
  invoice.discount_rate: 0.25
requires_approval: false
```
