# Fact Snapshot

A point-in-time capture of the facts used in a decision. Snapshots make decisions replayable and auditable — if a decision is challenged later, the snapshot provides the exact inputs that produced it.

## Shape

```yaml
primitive: fact_snapshot
as_of: <datetime_field>
facts:
  - <field_path>
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `as_of` | datetime field | yes | The moment at which facts were captured (typically `decision.created_at` or `now`) |
| `facts` | list of field paths | yes | The specific facts to capture; should include all inputs that influenced the outcome |

## Use when

- A decision may be reviewed, disputed, or audited after the fact
- Facts are mutable and may change after the decision is made (prices, statuses, balances)
- Regulatory or compliance requirements demand that decision inputs be retained
- You need to replay a decision with the same inputs to verify or debug it

## Composes with

- [fact](./fact.md) — the individual facts being captured
- [derived-fact](./derived-fact.md) — derived values should be snapshotted alongside source facts
- [audit-requirement](../governance/audit-requirement.md) — the snapshot is the primary artifact satisfying audit capture
- [decision-trace](../explanation/decision-trace.md) — the trace references the snapshot to explain which inputs drove which outcome
- [rule-version](../governance/rule-version.md) — the rule version active at `as_of` should be recorded alongside the snapshot

## Example

```yaml
primitive: fact_snapshot
as_of: decision.created_at
facts:
  - customer.risk_score
  - invoice.balance
  - account.status
  - invoice.days_past_due
```
