# Business Logic Primitives

Domain-neutral atoms for modeling business rules. Every pattern — eligibility check, SLA deadline, approval workflow, dunning sequence — is a composition of these primitives.

See [BLT.md](../BLT.md) for the full design rationale and worked examples.

## Architecture

```
Primitive   — reusable atom (this catalog)
Pattern     — named composition of primitives
Domain pack — industry-specific assembly of patterns
```

---

## Entity

What the rule is about.

| Primitive | Description |
|-----------|-------------|
| [Entity](./entity/entity.md) | A named business object (order, invoice, account) |
| [Actor](./entity/actor.md) | The person, system, or org performing an action |
| [Resource](./entity/resource.md) | Something scarce that can be reserved or consumed |
| [Relationship](./entity/relationship.md) | A named directional link between two entities |
| [Collection](./entity/collection.md) | A filtered set of instances for aggregation or ranking |

## Fact

What data the rule uses.

| Primitive | Description |
|-----------|-------------|
| [Fact](./fact/fact.md) | A single named input value with type and source |
| [Derived Fact](./fact/derived-fact.md) | A value computed from other facts |
| [Fact Source](./fact/fact-source.md) | Provenance, trust level, and freshness of data |
| [Fact Snapshot](./fact/fact-snapshot.md) | Point-in-time capture of inputs for audit and replay |
| [Missing Fact](./fact/missing-fact.md) | Explicit fallback when required data is absent |

## Condition

Truth tests the rule evaluates.

| Primitive | Description |
|-----------|-------------|
| [Predicate](./condition/predicate.md) | A named boolean expression |
| [Comparison](./condition/comparison.md) | Two values compared with an operator |
| [Logical Composition](./condition/logical-composition.md) | Combines conditions with all / any / none / not |
| [Quantifier](./condition/quantifier.md) | Tests how many members of a collection satisfy a condition |
| [Threshold](./condition/threshold.md) | Checks whether a value crosses a boundary |
| [Membership](./condition/membership.md) | Tests whether a value belongs to a set |
| [Pattern Match](./condition/pattern-match.md) | Validates a value against a regex or format |
| [Temporal Predicate](./condition/temporal-predicate.md) | A condition involving dates, durations, or windows |
| [Change Predicate](./condition/change-predicate.md) | Tests whether a field transitioned from one value to another |

## Decision

Judgment outputs the rule produces.

| Primitive | Description |
|-----------|-------------|
| [Boolean Decision](./decision/boolean-decision.md) | A yes/no outcome |
| [Enum Decision](./decision/enum-decision.md) | A named outcome from a fixed set of values |
| [Classification](./decision/classification.md) | Assigns an entity to a category or tier |
| [Selection](./decision/selection.md) | Picks one candidate from a set |
| [Ranking](./decision/ranking.md) | Orders candidates by a score |
| [Reasoned Decision](./decision/reasoned-decision.md) | A decision bundled with reason codes and evidence |

## Calculation

Numeric and monetary outputs.

| Primitive | Description |
|-----------|-------------|
| [Formula](./calculation/formula.md) | A deterministic expression over facts |
| [Rate Application](./calculation/rate-application.md) | Applies a rate to a base amount |
| [Table Lookup](./calculation/table-lookup.md) | Resolves a value from a keyed table |
| [Tier Lookup](./calculation/tier-lookup.md) | Resolves a value from tiered bands |
| [Aggregation](./calculation/aggregation.md) | Summarizes a collection into a single value |
| [Allocation](./calculation/allocation.md) | Distributes an amount across recipients |
| [Bound](./calculation/bound.md) | Enforces a minimum and/or maximum on a value |
| [Rounding](./calculation/rounding.md) | Standardizes numeric precision |
| [Proration](./calculation/proration.md) | Calculates a proportional amount over a period |

## State

Lifecycle behavior.

| Primitive | Description |
|-----------|-------------|
| [State](./state/state.md) | A named lifecycle position for an entity |
| [Transition](./state/transition.md) | Moves an entity from one state to another |
| [Invariant](./state/invariant.md) | A condition that must always hold true |
| [Terminal State](./state/terminal-state.md) | A state from which no further transitions are allowed |
| [Hold](./state/hold.md) | A named block placed on an entity's allowed actions |
| [Release](./state/release.md) | Removes a hold when conditions are met |

## Time

Temporal control.

| Primitive | Description |
|-----------|-------------|
| [Instant](./time/instant.md) | A specific point in time |
| [Duration](./time/duration.md) | A length of time |
| [Window](./time/window.md) | A bounded interval between two instants |
| [Deadline](./time/deadline.md) | A computed due date based on a start and duration |
| [Schedule](./time/schedule.md) | A recurring pattern that generates trigger instants |
| [Calendar Adjustment](./time/calendar-adjustment.md) | Shifts a date to respect business days or holidays |
| [Effective Period](./time/effective-period.md) | The date range during which a rule or value is active |
| [As-Of Evaluation](./time/as-of-evaluation.md) | Evaluates facts and rules at a specific historical moment |

## Event

What happened and what was requested.

| Primitive | Description |
|-----------|-------------|
| [Event](./event/event.md) | Something that occurred in the business domain |
| [Command](./event/command.md) | A requested action issued by an actor |
| [Trigger](./event/trigger.md) | The signal that causes a rule to fire |
| [Emitted Event](./event/emitted-event.md) | An event produced as an effect of a rule |
| [Idempotency Key](./event/idempotency-key.md) | Ensures an effect is applied exactly once |

## Effect

What the rule does.

| Primitive | Description |
|-----------|-------------|
| [Set Field](./effect/set-field.md) | Assigns a value to a field |
| [Create Record](./effect/create-record.md) | Creates a new entity instance |
| [Update Record](./effect/update-record.md) | Modifies an existing entity instance |
| [Block Action](./effect/block-action.md) | Prevents an action from proceeding |
| [Allow Action](./effect/allow-action.md) | Explicitly permits an action |
| [Notify](./effect/notify.md) | Sends a message to a recipient |
| [Assign](./effect/assign.md) | Routes work to a person, role, or queue |
| [Create Obligation](./effect/create-obligation.md) | Creates a required task or duty with a due date |
| [Grant Entitlement](./effect/grant-entitlement.md) | Gives a subject access to a feature or service |
| [Revoke Entitlement](./effect/revoke-entitlement.md) | Removes a subject's access |
| [Reserve Resource](./effect/reserve-resource.md) | Commits a quantity of a resource |
| [Release Resource](./effect/release-resource.md) | Returns a committed resource to the pool |
| [Post Ledger Entry](./effect/post-ledger-entry.md) | Records a financial debit/credit |

## Exception

Deviation and error handling.

| Primitive | Description |
|-----------|-------------|
| [Exception](./exception/exception.md) | A named condition that overrides the base rule outcome |
| [Override](./exception/override.md) | Replaces a base outcome with an authorized alternative |
| [Waiver](./exception/waiver.md) | Forgives a fee, requirement, or obligation |
| [Fallback](./exception/fallback.md) | The outcome when no other rule matches |
| [Retry](./exception/retry.md) | Re-attempts a failed action with backoff |
| [Compensation](./exception/compensation.md) | Reverses prior effects when a later step fails |
| [Reversal](./exception/reversal.md) | Undoes a specific applied event or transaction |

## Composition

Control flow for combining primitives.

| Primitive | Description |
|-----------|-------------|
| [Rule](./composition/rule.md) | The basic when/if/then unit |
| [Rule Set](./composition/rule-set.md) | A group of rules evaluated together |
| [Decision Table](./composition/decision-table.md) | Condition/outcome pairs in tabular form |
| [Pipeline](./composition/pipeline.md) | Sequential steps where each feeds the next |
| [Branch](./composition/branch.md) | Conditional fork between two paths |
| [Gate](./composition/gate.md) | A hold point that blocks progress until a condition is met |
| [Join](./composition/join.md) | Waits for multiple parallel steps to complete |
| [Precedence](./composition/precedence.md) | Defines which rule wins when multiple apply |
| [Conflict Resolver](./composition/conflict-resolver.md) | Detects and resolves contradictory outcomes |
| [Dependency](./composition/dependency.md) | Declares that one primitive requires another to be satisfied first |

## Governance

Control, versioning, and safe operation.

| Primitive | Description |
|-----------|-------------|
| [Policy Reference](./governance/policy-reference.md) | Links a rule to its authoritative source |
| [Rule Version](./governance/rule-version.md) | Tracks changes to a rule over time |
| [Approval Requirement](./governance/approval-requirement.md) | Requires human sign-off before an outcome takes effect |
| [Audit Requirement](./governance/audit-requirement.md) | Specifies what must be captured for traceability |
| [Retention Requirement](./governance/retention-requirement.md) | How long a record must be kept |
| [Privacy Constraint](./governance/privacy-constraint.md) | Restricts data use to permitted purposes |
| [Rollout Control](./governance/rollout-control.md) | Gradually releases a rule change to a subset of traffic |
| [Kill Switch](./governance/kill-switch.md) | Disables a rule immediately with a safe fallback |

## Explanation

Making decisions inspectable.

| Primitive | Description |
|-----------|-------------|
| [Reason Code](./explanation/reason-code.md) | A machine-readable code identifying why an outcome was produced |
| [Evidence](./explanation/evidence.md) | A specific fact value that influenced a decision |
| [Explanation Template](./explanation/explanation-template.md) | A human-readable message template for a decision |
| [Decision Trace](./explanation/decision-trace.md) | A full record of which rules fired and why |

## Observability

Making the rule system measurable.

| Primitive | Description |
|-----------|-------------|
| [Rule Execution Log](./observability/rule-execution-log.md) | Per-execution record of rule id, version, and result |
| [Metric](./observability/metric.md) | An aggregate measure computed from rule outcomes |
| [Alert](./observability/alert.md) | Fires when a metric crosses a threshold |
| [Drift Monitor](./observability/drift-monitor.md) | Detects when a metric shifts significantly from its baseline |
