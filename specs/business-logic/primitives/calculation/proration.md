# Proration

Scales an amount to the fraction of a billing period that was actually used or owed. Proration encodes the business rule for mid-period changes — upgrades, downgrades, cancellations, and seat additions — so the covered-period ratio and day-count basis are explicit rather than buried in billing code.

## Shape

```yaml
primitive: proration
amount: monthly_price
period: billing_period
covered_period:
  start: upgrade.effective_at
  end: billing_period.end
basis: calendar_days
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `amount` | field path or number | yes | The full-period amount to prorate |
| `period` | field path | yes | Reference to the full billing period (provides the total duration denominator) |
| `covered_period` | object | yes | The sub-period the customer actually used or is owed |
| `covered_period.start` | field path or date | yes | Inclusive start of the covered sub-period |
| `covered_period.end` | field path or date | yes | Inclusive end of the covered sub-period |
| `basis` | enum | yes | Day-count convention used to measure both periods: `calendar_days`, `business_days`, `thirty_360`, `actual_actual` |
| `target` | field path | no | If provided, writes the prorated result to this field |

**Basis options:**
- `calendar_days` — count every calendar day; most common for SaaS and subscription billing
- `business_days` — exclude weekends and holidays; used when service is only rendered on working days
- `thirty_360` — treat every month as 30 days and every year as 360; common in bond and lease calculations
- `actual_actual` — use exact day counts for both numerator and denominator; required for precise interest accrual

## Use when

- A customer upgrades, downgrades, or cancels mid-billing-period and must be charged only for days used
- A new seat or license is added partway through a period and requires a partial first-period charge
- A credit is issued for unused days remaining after a downgrade or cancellation
- Service fees or interest must be apportioned to a sub-period for accurate revenue recognition

## Composes with

- [formula](./formula.md) — compute the full-period amount that feeds into proration
- [rounding](./rounding.md) — round the prorated result to currency precision after the ratio is applied
- [bound](./bound.md) — ensure the prorated amount does not exceed the full-period charge or fall below zero
- [fact](../fact/fact.md) — effective dates, billing period boundaries, and prices are sourced as facts
- [entity/entity](../entity/entity.md) — the subscription or contract entity defines the period and price inputs

## Example

```yaml
primitive: proration
amount: monthly_price
period: billing_period
covered_period:
  start: upgrade.effective_at
  end: billing_period.end
basis: calendar_days
```

```yaml
primitive: proration
amount: annual_license_fee
period: contract_year
covered_period:
  start: contract_year.start
  end: cancellation.effective_at
basis: actual_actual
target: refund.prorated_amount
```
