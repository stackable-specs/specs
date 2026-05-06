# Aggregation

Reduces a filtered collection of records to a single summary value using a statistical function. Aggregation bridges the gap between individual records and account-level or period-level figures — open balances, invoice counts, average order values — that downstream rules and decisions depend on.

## Shape

```yaml
primitive: aggregation
collection: invoices
filter: invoice.status = Open
function: sum
field: invoice.balance
target: account.open_balance
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `collection` | string | yes | Name of the entity collection to aggregate over |
| `filter` | predicate expression | no | Condition that narrows which records are included; omit to aggregate all records |
| `function` | enum | yes | Aggregation function: `sum`, `count`, `average`, `min`, `max`, `median`, `percentile` |
| `field` | field path | yes (except `count`) | The field on each record to apply the function to; not required when `function` is `count` |
| `target` | field path | yes | The field that receives the aggregated result |
| `percentile` | number | no | Required when `function` is `percentile`; value between 0 and 100 |

## Use when

- You need a rolled-up figure across multiple records, such as total open balance or invoice count
- A threshold or eligibility rule requires knowing a cumulative or statistical value before firing
- A downstream `tier-lookup` or `rate-application` needs an aggregated quantity as its input
- You are computing period-based metrics such as monthly spend, average days to pay, or peak usage

## Composes with

- [tier-lookup](./tier-lookup.md) — aggregated quantity drives tier selection for volume pricing
- [formula](./formula.md) — aggregation result can be an operand in a larger formula
- [rate-application](./rate-application.md) — apply a rate to the aggregated base amount
- [entity/collection](../entity/collection.md) — defines the collection and filter scope being aggregated
- [condition/threshold](../condition/threshold.md) — compare the aggregated result against a threshold to trigger an action

## Example

```yaml
primitive: aggregation
collection: invoices
filter: invoice.status = Open
function: sum
field: invoice.balance
target: account.open_balance
```

```yaml
primitive: aggregation
collection: orders
filter: order.placed_at >= period.start AND order.placed_at <= period.end
function: count
target: account.orders_this_period
```
