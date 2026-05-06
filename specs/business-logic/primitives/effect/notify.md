# Notify

Sends a message to a recipient via a specified channel using a named template. The primitive captures the intent to notify — who receives the message, how it is delivered, and which template governs its content — without embedding the message text in the rule itself. Template rendering and delivery mechanics are handled by the notification service; the rule only declares the signal.

## Shape

```yaml
primitive: notify
recipient: customer.email
channel: email
template: payment_failed_notice
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `recipient` | field path or literal | yes | Address or identifier of the person or system to notify; field paths resolve at runtime (e.g., `customer.email`, `account.owner_id`) |
| `channel` | enum | yes | Delivery mechanism: `email`, `sms`, `push`, `in_app`, `webhook` |
| `template` | string | yes | Snake-case name of the message template to render; templates are managed outside the rule catalog |
| `data` | map | no | Additional field paths or literals to pass as template variables at render time |
| `delay` | duration expression | no | If set, the notification is scheduled for delivery after the specified delay rather than immediately |
| `deduplicate_within` | duration expression | no | Suppresses duplicate notifications for the same recipient and template within the given window |

## Use when

- A rule outcome must alert a customer, agent, or internal team (e.g., notify the customer when a payment fails, notify the collections queue when an invoice is 30 days overdue)
- A state transition needs to produce a human-readable confirmation or warning via a communication channel
- A compliance rule requires that affected parties be informed within a specific time window
- You want to decouple notification content (owned by marketing or ops) from business rule logic (owned by engineering or product)
- Delivery channel selection is driven by contact preferences stored on the entity (e.g., use SMS if `customer.sms_opt_in` is true)

## Composes with

- [event](../event/event.md) — a domain event commonly triggers a `notify` effect; the event carries the context, the notify carries the communication
- [create-obligation](./create-obligation.md) — when a notification requires a human response, pair it with an obligation that tracks the response deadline
- [block-action](./block-action.md) — when an action is blocked, a `notify` effect surfaces the block reason to the affected party
- [entity](../entity/entity.md) — the recipient field path resolves through the subject entity and its relationships (e.g., `order.customer.email`)
- [fact](../fact/fact.md) — template data fields are often derived facts computed earlier in the rule chain

## Example

```yaml
primitive: notify
recipient: customer.email
channel: email
template: payment_failed_notice
data:
  invoice_number: invoice.number
  amount_due: invoice.balance
  due_date: invoice.due_date
```

```yaml
primitive: notify
recipient: support_team.webhook_url
channel: webhook
template: high_value_churn_risk_alert
data:
  account_id: account.id
  mrr: account.monthly_recurring_revenue
  days_since_last_login: account.days_since_last_login
deduplicate_within: 24h
```
