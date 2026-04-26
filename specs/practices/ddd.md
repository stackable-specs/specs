---
id: ddd
layer: practices
extends: []
---

# Domain-Driven Design (DDD)

## Purpose

Software complexity that lives in the wrong place — domain rules buried in HTTP controllers, invariants enforced by database triggers, terms whose meaning shifts as you cross from "billing" code to "checkout" code — produces systems that work in any single workflow but fail catastrophically when teams cross-cut. DDD names the discipline that prevents this: a single ubiquitous language per bounded context, aggregates that own their invariants, dependencies that point inward toward the domain, and explicit translation at context boundaries. Without those constraints, codebases accumulate anemic data classes, generic `Manager` / `Helper` services, transactions that span unrelated entities, and integrations that copy each other's data shapes — none of which fail loudly until a feature touches several boundaries at once. This spec pins the strategic and tactical DDD patterns so the code's structure carries the domain's structure, language stays consistent within a context, and the domain layer does not silently take dependencies on framework or persistence concerns.

## References

- **spec** `madr` — record DDD-shaping decisions (chosen contexts, integration patterns) as ADRs
- **external** `https://en.wikipedia.org/wiki/Domain-driven_design` — Domain-Driven Design overview
- **external** `https://martinfowler.com/bliki/DomainDrivenDesign.html` — Martin Fowler on DDD
- **external** `https://martinfowler.com/bliki/BoundedContext.html` — Bounded Context
- **external** `https://www.domainlanguage.com/ddd/reference/` — Eric Evans, *DDD Reference* (free PDF)

## Rules

1. Identify and document the project's core domain explicitly, distinguishing it from supporting and generic subdomains; do not treat every part of the system as equally important to invest in.
2. Define each bounded context with an explicit name and a documented scope; commit a context inventory in version control.
3. Maintain a context map describing the relationships between bounded contexts (shared kernel, customer / supplier, conformist, anti-corruption layer, partnership, separate ways, open host service, published language).
4. Use one ubiquitous language per bounded context; do not let terms from another context appear in this context's code, tests, or documentation without translation.
5. Translate terms at context boundaries through an anti-corruption layer when integrating with a context whose language differs; do not let a foreign model's vocabulary propagate into the domain.
6. Distinguish entities (identity-based equality, lifecycle) from value objects (value-based equality, immutability) explicitly in code and naming.
7. Make value objects immutable; do not expose setters or mutating methods on them.
8. Group entities and value objects into aggregates with a single root entity that owns access to all members; do not let external code reach internal entities directly.
9. Reference other aggregates only by their root's identity (ID), not by holding direct references to their internal members.
10. Modify a single aggregate per transaction; do not commit changes spanning multiple aggregates atomically when eventual consistency is acceptable.
11. Construct aggregates through factories or root constructors that enforce invariants at creation time; do not allow direct field-level instantiation that can produce invalid state.
12. Encapsulate persistence behind repository interfaces that return whole aggregates; do not let domain code depend on ORM types or database client APIs.
13. Place repository interfaces in the domain layer and their implementations in the infrastructure layer.
14. Place business rules and invariants on the entities and value objects they constrain; do not extract them into services that exist only to operate on anemic data structures.
15. Use domain services only for operations that do not naturally belong to a single entity or value object.
16. Keep application services thin: orchestrate aggregates, coordinate transactions, and translate inputs — do not enforce domain rules at the application layer.
17. Publish domain events from aggregates when a state change is meaningful outside the aggregate; do not encode cross-aggregate coordination as direct method calls between aggregates.
18. Name domain events as past-tense business facts (`OrderPlaced`, `PaymentReceived`, `InventoryReserved`); treat the published payload as immutable.
19. Make the domain layer depend on nothing outside itself; UI / interface, application, and infrastructure layers depend on the domain, not the reverse.
20. Do not import infrastructure types (database clients, HTTP frameworks, message broker SDKs) into domain-layer code.
21. Name classes, methods, modules, and database columns using the bounded context's ubiquitous language; do not introduce generic terms (`Manager`, `Helper`, `Util`, `Data`) where a domain term exists.
22. Update the ubiquitous-language documentation in the same pull request that introduces or renames a domain term in the code; do not let documentation and code drift.
