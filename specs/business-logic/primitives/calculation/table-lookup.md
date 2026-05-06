# Table Lookup

Resolves a value from a named reference table by matching one or more key fields. Table lookup externalizes data-driven decisions — tax rates, fee schedules, discount matrices — out of rule expressions and into maintainable tables, so rates change without touching rule logic.

## Shape

```yaml
primitive: table_lookup
table: tax_rates
keys:
  jurisdiction: order.ship_to.jurisdiction
  product_tax_code: line_item.tax_code
result: tax_rate
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `table` | string | yes | Name of the reference table to query |
| `keys` | map of string → field path | yes | One or more column-to-field mappings used to locate the matching row |
| `result` | string | yes | The column name whose value is returned from the matched row |

## Use when

- A rate or value depends on a combination of attributes that form a matrix (jurisdiction × product type, customer tier × volume band)
- The underlying data changes frequently and must be updated without modifying rule definitions
- Multiple rules need to draw from the same authoritative rate table
- You want a clean separation between rule structure and reference data

## Composes with

- [rate-application](./rate-application.md) — the looked-up rate becomes the multiplier for a base amount
- [tier-lookup](./tier-lookup.md) — use when the key is a continuous range rather than discrete values
- [fact](../fact/fact.md) — key fields are typically facts sourced from an entity
- [entity/collection](../entity/collection.md) — iterate a collection and apply a lookup per row
- [condition/predicate](../condition/predicate.md) — guard the lookup with a condition before executing

## Example

```yaml
primitive: table_lookup
table: tax_rates
keys:
  jurisdiction: order.ship_to.jurisdiction
  product_tax_code: line_item.tax_code
result: tax_rate
```

```yaml
primitive: table_lookup
table: carrier_surcharges
keys:
  carrier_code: shipment.carrier
  service_level: shipment.service_level
result: fuel_surcharge_rate
```
