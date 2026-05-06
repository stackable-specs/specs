# Entity

A named business object that rules are written about. The entity is the subject of predicates, decisions, state transitions, and effects — the "thing" the rule operates on.

## Shape

```yaml
primitive: entity
name: <string>
id_field: <field_path>
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Pascal-case name of the business object |
| `id_field` | field path | yes | The field that uniquely identifies an instance |

## Use when

- You are writing a rule that targets a specific business object (order, invoice, account, claim, subscription)
- You need to declare the subject so downstream primitives — predicates, transitions, effects — can reference it unambiguously
- Multiple entity types appear in the same rule and you need to distinguish them

## Composes with

- [actor](./actor.md) — the entity performing an action, as distinct from the entity being acted on
- [relationship](./relationship.md) — connects this entity to others
- [collection](./collection.md) — a filtered set of this entity type
- [fact](../fact/fact.md) — facts are attributes sourced from an entity

## Example

```yaml
primitive: entity
name: Invoice
id_field: invoice.id
```

```yaml
primitive: entity
name: Subscription
id_field: subscription.id
```
