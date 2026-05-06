# Ranking

A rule that orders a candidate set by a score expression and produces a sorted list. Ranking makes the prioritization logic explicit and composable — downstream rules can slice, page, or select from the ordered result without re-implementing the sort.

## Shape

```yaml
primitive: ranking
candidates: tickets
score: ticket.priority_score
order: descending
tie_breakers:
  - ticket.created_at ascending
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `candidates` | collection name or field path | yes | The set of items to rank; must resolve to an iterable collection |
| `score` | field path or expression | yes | The value used as the primary sort key; can reference a computed fact or a raw field |
| `order` | `ascending` or `descending` | yes | Direction of the primary sort; `descending` puts the highest score first |
| `tie_breakers` | tie-breaker list | no | Ordered list of secondary sort criteria applied when `score` values are equal; each entry is a field path followed by `ascending` or `descending` |

## Use when

- A set of items must be presented or processed in priority order (work queues, recommendation feeds, search results)
- The sort key is a computed or derived value that should be declared centrally rather than repeated across callers
- Tie-breaking rules carry business meaning and need to be versioned independently of the primary score
- Downstream logic (e.g. selection, paging, batching) consumes the ranked output rather than re-sorting
- Audit or explanation requires a record of which score each candidate received and what position it occupied

## Composes with

- [selection](./selection.md) — selection frequently operates on the top of a ranked list, picking the single winner after ranking has ordered the field
- [fact](../fact/fact.md) — facts compute or surface the score values that ranking evaluates
- [classification](./classification.md) — classify candidates before ranking to restrict the ranked set to a single tier or category
- [predicate](../condition/predicate.md) — add a pre-filter step using predicates to exclude ineligible candidates before the ranking score is applied

## Example

```yaml
primitive: ranking
candidates: tickets
score: ticket.priority_score
order: descending
tie_breakers:
  - ticket.created_at ascending
```

```yaml
primitive: ranking
candidates: loan_applications
score: applicant.credit_score * 0.6 + application.loan_to_value_ratio * 0.4
order: descending
tie_breakers:
  - application.submitted_at ascending
  - application.requested_amount ascending
```
