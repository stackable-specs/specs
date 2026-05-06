# Relationship

A named, directional link between two entities with optional cardinality constraints. Relationships let rules reason about structure — ownership, membership, hierarchy, authorization scope — without embedding that structure inside a single entity.

## Shape

```yaml
primitive: relationship
subject: <entity>
relation: <string>
object: <entity>
cardinality:
  max_per_subject: <number>
  max_per_object: <number>
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `subject` | entity name | yes | The entity holding the relationship |
| `relation` | string | yes | The named link (e.g. `PrimaryContactOf`, `ManagedBy`, `BelongsTo`) |
| `object` | entity name | yes | The entity being related to |
| `cardinality.max_per_subject` | number | no | Maximum number of objects a single subject may relate to |
| `cardinality.max_per_object` | number | no | Maximum number of subjects that may relate to a single object |

## Use when

- A rule's conditions depend on how two entities are connected, not just their individual attributes
- Cardinality must be enforced (one primary contact per account, one active subscription per product per customer)
- Authorization scope is defined by ownership or membership (a user may only edit records they own)
- Hierarchy or delegation needs to be traversed (manager of a manager)

## Composes with

- [entity](./entity.md) — both subject and object are entities
- [predicate](../condition/predicate.md) — rules test whether a relationship exists
- [membership](../condition/membership.md) — membership in a set is a common relationship pattern
- [cardinality-constraint](../composition/rule.md) — enforces the min/max counts declared here

## Example

```yaml
primitive: relationship
subject: Person
relation: PrimaryContactOf
object: Account
cardinality:
  max_per_object: 1
```

```yaml
primitive: relationship
subject: User
relation: OwnedBy
object: Contract
cardinality:
  max_per_subject: 100
```
