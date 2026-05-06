# Business Logic Templates (BLT)

A domain-neutral library for modeling business rules. Every eligibility check, pricing calculation, approval workflow, or SLA policy is expressed as a composition of named primitives — no prose, no ambiguity, no implementation assumptions.

## What it is

BLT specs are policy artifacts, not code. They sit between a business requirement and an implementation: precise enough for an engineer to build from directly, readable enough for a product manager or analyst to validate.

The unit of authoring is the **rule**:

```
WHEN  <trigger>
GIVEN <facts>
IF    <conditions>
THEN  <effects>
```

Complex processes are compositions of rules using pipelines, gates, branches, and rule sets.

## Contents

```
specs/business-logic/
  primitives/     — reusable atoms: entities, facts, conditions, effects, and more
  practices/      — authoring workflow, naming conventions, testing, versioning
```

| Document | Purpose |
|----------|---------|
| [Primitive Catalog](./primitives/README.md) | Full reference of every primitive type, organized by group |
| [BLT Practices](./practices/blt.md) | Step-by-step authoring guide, composition patterns, common mistakes |

## Architecture

```
Primitive   — reusable atom (specs/business-logic/primitives/)
Pattern     — named composition of primitives
Domain pack — industry-specific assembly of patterns
```

Primitives are the vocabulary. Patterns are the sentences. Domain packs are documents in that vocabulary applied to a specific industry or problem space.

## When to use BLT

Use BLT when a business requirement involves branching logic, configurable thresholds, exceptions, audit requirements, or outcomes that must be explainable. If the logic can be expressed as a single formula with no branching, a plain spec is sufficient.

## Quick start

1. Read [BLT Practices](./practices/blt.md) for the authoring workflow.
2. Browse the [Primitive Catalog](./primitives/README.md) to find the building blocks you need.
3. Author a spec using the `WHEN / GIVEN / IF / THEN` structure and reference primitives by name.
4. Include at minimum: happy path, negative case, exception case, and missing-fact case in `tests:`.
