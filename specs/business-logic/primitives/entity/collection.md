# Collection

A filtered set of entity instances used as the input to aggregation, ranking, allocation, quantification, or iteration. A collection is not a static list — it is a query: a named, reusable view over an entity type.

## Shape

```yaml
primitive: collection
name: <string>
entity: <entity>
filter:
  - <predicate>
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | A descriptive name for this filtered set (e.g. `OpenInvoices`, `EligibleWarehouses`) |
| `entity` | entity name | yes | The entity type this collection contains |
| `filter` | list of predicates | no | Conditions that every member of the set must satisfy; omit to include all instances |

## Use when

- A rule needs to operate over multiple records rather than a single instance (sum, count, find, rank, allocate)
- A quantifier needs a set to evaluate (all line items must have a tax code)
- An allocation needs a pool of candidates (apply payment to open invoices)
- A selection rule needs a candidate set (choose the lowest-cost eligible warehouse)

## Composes with

- [aggregation](../calculation/aggregation.md) — sums, counts, or averages values across the collection
- [quantifier](../condition/quantifier.md) — tests whether all, any, or none of the members satisfy a condition
- [allocation](../calculation/allocation.md) — distributes an amount across members
- [ranking](../decision/ranking.md) — orders members by a score
- [selection](../decision/selection.md) — picks one member from the set

## Example

```yaml
primitive: collection
name: OpenInvoices
entity: Invoice
filter:
  - invoice.status = Open
```

```yaml
primitive: collection
name: EligibleWarehouses
entity: Warehouse
filter:
  - warehouse.active = true
  - warehouse.region = order.shipping_region
```
