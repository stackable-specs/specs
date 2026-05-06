# Boolean Decision

A rule that resolves to either true or false for a given subject. Boolean decisions encode a binary business judgment — the outcome is unambiguous and downstream logic branches on it without further interpretation.

## Shape

```yaml
primitive: boolean_decision
name: refund_allowed
true_when:
  - <conditions>
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Snake-case name of the decision, phrased as a predicate (e.g. `refund_allowed`, `discount_eligible`) |
| `true_when` | condition list | yes | One or more conditions that must all hold for the decision to resolve `true`; if any condition fails the result is `false` |

## Use when

- A business rule has exactly two possible outcomes and no intermediate states
- Downstream logic gates an action on a yes/no judgment (e.g. "only send invoice if `payment_overdue` is true")
- You want to name and reuse a compound predicate across multiple rules without duplicating the conditions
- An audit trail needs a named, traceable true/false result rather than an inline expression
- The decision must be testable in isolation with a clear pass/fail expectation

## Composes with

- [predicate](../condition/predicate.md) — each entry in `true_when` is typically a predicate evaluated against entity fields
- [fact](../fact/fact.md) — facts supply the field values that conditions are evaluated against
- [reasoned-decision](./reasoned-decision.md) — wrap a boolean decision with reason codes when the caller needs to know *why* the result is true or false
- [enum-decision](./enum-decision.md) — use instead when the outcome space has more than two named values

## Example

```yaml
primitive: boolean_decision
name: refund_allowed
true_when:
  - order.status = Delivered
  - order.delivered_at > now() - 30d
  - item.final_sale = false
```

```yaml
primitive: boolean_decision
name: account_past_due
true_when:
  - invoice.due_date < today()
  - invoice.balance_due > 0
```
