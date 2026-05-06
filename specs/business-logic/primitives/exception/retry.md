# Retry

Re-attempts a failed action according to a controlled backoff policy. Retries are not silent loops — they are declared policies that specify how many times to try, how long to wait between attempts, and what to do when all attempts are exhausted. A retry policy makes failure handling explicit and auditable.

## Shape

```yaml
primitive: retry
action: charge_payment_method
max_attempts: 3
backoff: exponential
interval: 1 hour
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `action` | string | yes | The name of the action or effect to re-attempt |
| `max_attempts` | integer | yes | The total number of times to try, including the first attempt |
| `backoff` | enum | yes | The wait-time strategy between attempts: `fixed`, `exponential`, or `linear` |
| `interval` | duration | yes | The base wait duration; interpretation depends on the backoff type |
| `on_exhausted` | string or outcome map | no | The outcome or action to take when all attempts fail (e.g. emit an event, escalate, trigger compensation) |
| `retry_on` | list of error codes or conditions | no | Restrict retries to specific failure types; non-matching failures fail immediately |
| `jitter` | boolean | no | If true, adds random variance to the interval to prevent thundering-herd retry collisions |

## Backoff types

| Type | Behavior |
|------|----------|
| `fixed` | Waits exactly `interval` between every attempt |
| `exponential` | Doubles the wait time after each attempt: interval, 2x, 4x, 8x... |
| `linear` | Adds `interval` after each attempt: interval, 2x interval, 3x interval... |

## Use when

- A transient external failure (network timeout, payment gateway unavailability) may succeed on a subsequent attempt
- A downstream service is rate-limited and the action should be spaced out rather than abandoned immediately
- A workflow step must succeed eventually and the business can tolerate a delay before escalating
- You need a defined upper bound on retries so failures do not loop indefinitely

## Composes with

- [idempotency-key](../event/idempotency-key.md) — retries must pair with an idempotency key to ensure re-attempts do not duplicate effects
- [compensation](./compensation.md) — when retries are exhausted, compensation reverses effects already applied before the failure
- [emitted-event](../event/emitted-event.md) — emit a `RetryExhausted` or `ActionFailed` event when `on_exhausted` fires
- [exception](./exception.md) — an exception can short-circuit retries for known-unrecoverable failure conditions

## Example

```yaml
primitive: retry
action: charge_payment_method
max_attempts: 3
backoff: exponential
interval: 1 hour
jitter: true
retry_on:
  - gateway_timeout
  - insufficient_funds_transient
on_exhausted:
  emit:
    name: PaymentRetryExhausted
    payload:
      payment_id: payment.id
      final_failure_reason: payment.last_error
```

```yaml
primitive: retry
action: send_invoice_email
max_attempts: 5
backoff: linear
interval: 15 minutes
retry_on:
  - smtp_timeout
  - rate_limited
on_exhausted: escalate_to_support_queue
```
