# Reason Code

A machine-readable code identifying why an outcome was produced. Reason codes make decisions explainable to both systems and humans by attaching a stable, structured identifier to every outcome — enabling downstream systems to branch on the reason, and enabling UIs and agents to render a meaningful explanation to the person affected.

## Shape

```yaml
primitive: reason_code
code: OutsideReturnWindow
message: Order is outside the 30-day return window.
severity: blocking
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `code` | string | yes | PascalCase machine-readable identifier for the reason. Stable across versions and safe for downstream branching |
| `message` | string | yes | Default human-readable explanation of the reason, suitable for display in a UI or log |
| `severity` | enum | yes | How strongly this reason affects the outcome. One of: `blocking` (prevents the outcome), `warning` (allows the outcome with a flag), `informational` (recorded for context only) |
| `category` | string | no | Grouping label for related reason codes (e.g., `eligibility`, `fraud`, `pricing`, `compliance`) |
| `documentation_url` | string | no | Link to internal documentation explaining what triggers this code and how to resolve it |

## Use when

- A rule produces an outcome (approval, denial, flag, escalation) and a downstream system or user needs to know the specific reason
- Multiple rules can produce the same outcome type (e.g., "denied") for different reasons and callers must distinguish them
- You want to enable callers to programmatically react to specific decision outcomes without parsing free-text messages
- A customer, agent, or regulator will need to see a plain-language explanation of why a decision was made
- You are composing an `explanation-template` that assembles reason codes into a structured message

## Composes with

- [explanation-template](./explanation-template.md) — explanation templates interpolate reason codes into audience-specific messages; the `code` field is the key used for interpolation
- [evidence](./evidence.md) — reason codes explain what happened; evidence records the specific fact values that caused it — together they form a complete explanation
- [decision-trace](./decision-trace.md) — the decision trace lists all reason codes produced during evaluation alongside the rules and conditions that generated them
- [../governance/audit-requirement](../governance/audit-requirement.md) — reason codes are a standard capture target in audit requirements, making log entries self-explanatory

## Example

```yaml
primitive: reason_code
code: OutsideReturnWindow
message: Order is outside the 30-day return window.
severity: blocking
category: eligibility
documentation_url: https://wiki.internal/refunds/reason-codes#OutsideReturnWindow
```

```yaml
primitive: reason_code
code: InsufficientAccountBalance
message: Account balance is below the minimum required for this transaction.
severity: blocking
category: fraud
```

```yaml
primitive: reason_code
code: LoyaltyDiscountApplied
message: A loyalty discount was applied because the customer is a Gold tier member.
severity: informational
category: pricing
```
