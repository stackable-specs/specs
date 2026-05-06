# Selection

A rule that chooses one item from a candidate set according to a declared mode and optional constraints. Selection externalizes the "pick the best option" logic that would otherwise be buried in application code, making the strategy explicit, auditable, and replaceable.

## Shape

```yaml
primitive: selection
candidates: warehouses
mode: lowest_cost
constraints:
  - warehouse.has_inventory = true
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `candidates` | collection name or field path | yes | The set of items to choose from; must resolve to an iterable collection |
| `mode` | enum | yes | The selection strategy to apply after constraints are enforced (see modes below) |
| `constraints` | condition list | no | Filters applied before the mode logic; candidates that fail any constraint are excluded from consideration |

### Modes

| Mode | Behavior |
|------|----------|
| `lowest_cost` | Selects the candidate with the minimum value on the relevant cost field |
| `highest_value` | Selects the candidate with the maximum value on the relevant value field |
| `first_match` | Selects the first candidate that passes all constraints in collection order |
| `best_score` | Selects the candidate with the highest composite score (requires a `score` field or expression) |
| `round_robin` | Distributes selections evenly across candidates in rotation |

## Use when

- Business logic must choose exactly one item from a dynamic set (warehouse, carrier, agent, vendor)
- The selection strategy is a named policy that may change independently of surrounding rules
- Constraints that disqualify candidates need to be version-controlled and audited separately from the mode
- Multiple rules share the same candidate pool but apply different modes or constraints
- You need to explain after the fact which candidate was chosen and why it won

## Composes with

- [ranking](./ranking.md) — use ranking when you need an ordered list of candidates rather than a single winner; selection is often the final step after a ranking pass
- [predicate](../condition/predicate.md) — each entry in `constraints` is a predicate evaluated against candidate fields
- [fact](../fact/fact.md) — facts expose the field values (cost, capacity, score) that the mode logic compares
- [boolean-decision](./boolean-decision.md) — pair with a boolean decision to gate whether selection should run at all

## Example

```yaml
primitive: selection
candidates: warehouses
mode: lowest_cost
constraints:
  - warehouse.has_inventory = true
  - warehouse.ships_to = order.destination_country
```

```yaml
primitive: selection
candidates: support_agents
mode: round_robin
constraints:
  - agent.status = Available
  - agent.queue_depth < 10
  - agent.languages includes ticket.language
```
