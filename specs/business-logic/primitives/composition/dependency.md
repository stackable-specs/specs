# Dependency

Declares that one rule, entitlement, or feature requires another to be satisfied or active before it can apply. A dependency is a prerequisite relationship, not a sequencing relationship — it does not say "run A then B," it says "B is only valid when A is already true." This keeps the dependent rule clean of prerequisite logic and makes the dependency graph inspectable.

## Shape

```yaml
primitive: dependency
target: AdvancedAnalytics
requires:
  - BasicAnalytics
  - plan in [Pro, Enterprise]
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `target` | string | yes | The rule, entitlement, or feature that has a prerequisite |
| `requires` | list | yes | One or more rules, entitlements, or conditions that must all be satisfied before `target` can apply |
| `unsatisfied_behavior` | enum | no | What happens when prerequisites are not met; defaults to `block` |
| `message` | string | no | Human-readable explanation shown when the dependency is not satisfied |

### `unsatisfied_behavior` values

| Value | Meaning |
|-------|---------|
| `block` | Prevent the target from applying; surface an error or validation message (default) |
| `auto_satisfy` | Automatically activate the prerequisite rules if they are not already active |
| `warn` | Allow the target to apply but emit a warning |
| `escalate` | Route to a human reviewer to decide whether to override the dependency |

## Use when

- A pricing tier, feature flag, or entitlement only makes sense if a foundational entitlement is already active (e.g. add-ons that require a base plan)
- A rule references data or state that is only produced by another rule, making the dependency implicit — a dependency primitive makes it explicit
- You are modelling product bundling constraints where certain SKUs can only be purchased alongside or after another SKU
- You need to validate that a configuration is self-consistent before applying it — missing prerequisites surface as actionable errors rather than runtime failures
- An approval workflow step requires a prior step to have been completed and signed off

## Composes with

- [rule](./rule.md) — the `target` and each item in `requires` typically reference named rules
- [rule-set](./rule-set.md) — a rule set with `all_must_pass` mode can enforce a dependency group, but a dependency primitive makes the relationship named and inspectable
- [gate](./gate.md) — a gate is a runtime hold point; a dependency is a structural prerequisite — they are complementary
- [pipeline](./pipeline.md) — pipeline step ordering encodes sequencing; a dependency encodes a logical prerequisite that exists regardless of order
- [entity](../entity/entity.md) — the `target` and `requires` fields often reference entitlements or features attached to an entity such as a Subscription or Account

## Example

```yaml
primitive: dependency
target: AdvancedAnalytics
requires:
  - BasicAnalytics
  - plan in [Pro, Enterprise]
unsatisfied_behavior: block
message: "Advanced Analytics requires Basic Analytics to be active and a Pro or Enterprise plan."
```

```yaml
primitive: dependency
target: MultiCurrencyPricing
requires:
  - BasePricingEngine
  - feature_flag.currency_conversion = enabled
  - account.verified_bank_account = true
unsatisfied_behavior: auto_satisfy
```

```yaml
primitive: dependency
target: HighValueOrderApproval
requires:
  - CreditCheckPassed
  - FraudScreeningPassed
unsatisfied_behavior: escalate
message: "High-value order approval requires both credit check and fraud screening to have passed."
```
