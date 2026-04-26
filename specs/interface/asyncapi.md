---
id: asyncapi
layer: interface
extends: []
---

# AsyncAPI

## Purpose

AsyncAPI is the contract that lets a publisher team, a subscriber team, an SDK generator, a broker operator, a schema registry, and a docs site all talk about the same event-driven API without reading each other's code — but only if the description is the single source of truth, kept in version control, linted, and diff-gated for breaking changes. When teams treat AsyncAPI as documentation generated *from* code annotations as an afterthought, the description drifts: a channel address is renamed without bumping `info.version`, a required header silently becomes optional, an enum value disappears mid-release, a producer changes payload shape and the contract still claims the old one, and every consumer discovers the change by way of a deserialization failure or a poison-message loop. When teams skip examples, omit `operationId`, mix request/reply with pub/sub on the same operation, leave bindings out, and inline schemas instead of `$ref`-ing components, generated clients are unusable and reviewers cannot tell which changes are breaking. This spec pins the AsyncAPI version, the design-first workflow, the version-control and CI gates (lint + breaking-change diff), the structural conventions (`channels`, `operations`, `messages`, `components`, `$ref`, naming), the protocol bindings (Kafka, AMQP, MQTT, NATS, WebSocket, HTTP) needed to make the document executable, the security and server shape, and the code generation flow — so the AsyncAPI document is a contract two unrelated teams can build against, not a stale picture of last sprint's events.

## References

- **spec** `openapi` — synchronous-API counterpart with the same design-first workflow
- **spec** `kafka` — a common transport whose channels and bindings AsyncAPI describes
- **spec** `nats` — a common transport whose channels and bindings AsyncAPI describes
- **spec** `rabbitmq` — a common transport whose channels and bindings AsyncAPI describes
- **external** `https://www.asyncapi.com/` — AsyncAPI Initiative
- **external** `https://www.asyncapi.com/docs/reference/specification/latest` — AsyncAPI Specification (latest)
- **external** `https://www.asyncapi.com/docs/concepts` — AsyncAPI concepts
- **external** `https://github.com/asyncapi/bindings` — Protocol bindings (Kafka, AMQP, MQTT, NATS, WebSocket, HTTP, …)
- **external** `https://github.com/stoplightio/spectral` — Spectral linter (ships AsyncAPI rulesets)
- **external** `https://github.com/asyncapi/diff` — AsyncAPI diff / breaking-change tool
- **external** `https://github.com/asyncapi/generator` — AsyncAPI Generator (templates for server / client / docs)
- **external** `https://json-schema.org/specification.html` — JSON Schema (referenced by AsyncAPI payload schemas)
- **external** `https://www.cloudevents.io/` — CloudEvents (a common envelope for AsyncAPI messages)

## Rules

1. Author and version-control an AsyncAPI document (`.yaml` or `.json`) for every event-driven or message-based API the team owns; the AsyncAPI document is the single source of truth, not code annotations, wikis, or generated HTML.
2. Use AsyncAPI 3.x for new APIs; accept 2.x only when a required downstream tool does not yet support 3.x, and document the constraint.
3. Adopt a design-first workflow — change the AsyncAPI document first, then generate or update producer, consumer, and docs artifacts; do not let code annotations be the de facto source from which a "current" document is regenerated.
4. Commit the AsyncAPI document to source control alongside the implementation that produces or consumes it; do not host the document only as a Confluence page or a generated HTML artifact.
5. Lint the AsyncAPI document in CI with Spectral (or equivalent) using a committed ruleset; treat lint errors as build failures.
6. Run an AsyncAPI breaking-change diff (`asyncapi diff` or equivalent) in CI against the previously released document; treat any breaking change as a build failure unless the major version is bumped in the same change.
7. Set `info.version` to a SemVer string (`MAJOR.MINOR.PATCH`) and bump it in the same commit that changes the API surface; do not edit the document without bumping `info.version`.
8. Bump `info.version`'s major component for every breaking change (removed channel, renamed operation, changed required payload field, narrowed type, removed enum value, changed channel address); do not slip a breaking change into a minor or patch bump.
9. Separate `channels` (transport addresses) from `operations` (the application's send/receive intent) per the AsyncAPI 3 model; do not collapse the two by overloading channel definitions with operation semantics.
10. Assign every operation an explicit `action` of `send` or `receive` from the document owner's perspective and a unique, stable `operationId` (camelCase or kebab-case per a single project convention); do not rename `operationId`s on cosmetic changes.
11. Express channel addresses with explicit, named address parameters (e.g. `user/{userId}/signedup`) and document each parameter under `channels.<id>.parameters` with `description` and `schema`; do not use ad-hoc, undocumented address segments.
12. Define every message under `components.messages` with a referenced `payload` schema, a `headers` schema where headers are used, a `contentType`, and at least one `examples` entry; do not ship a message whose `payload` is an inlined anonymous schema.
13. Define reusable schemas, parameters, messages, headers, server variables, security schemes, and operation traits under `components/*` and reference them with `$ref`; do not inline an artifact that is used by more than one channel or operation.
14. Name component schemas in `PascalCase`, message names in `PascalCase`, parameters in `camelCase`, channel ids and operation ids in `camelCase` (or a single project-wide convention), and tags in human-readable Title Case.
15. Tag every operation with at least one `tags` entry that maps to a documented tag in the top-level `tags` array; do not leave operations untagged.
16. Provide a `summary` (short verb phrase, e.g. "Publish order created", "Consume payment captured") and a `description` (long, Markdown) on every operation; the `summary` must fit in a navigation tree.
17. Declare every server under `servers` with an explicit `host`, `protocol`, `protocolVersion` (where applicable), and protocol-specific `bindings`; do not ship a document with no `servers` or with placeholder `host: example.com` values.
18. Declare protocol-specific `bindings` on the channel, operation, and message wherever the transport requires them (Kafka topic configs, AMQP exchange/queue, MQTT QoS, NATS subjects, WebSocket sub-protocols); do not omit bindings on a document targeted at a real broker.
19. Define the message envelope (CloudEvents, a project-defined wrapper, or a raw payload) once under `components.messages` and reference it from every operation that uses it; do not redefine the envelope per channel.
20. Declare every authentication mechanism under `components.securitySchemes` and apply it via top-level `security` or per-server / per-operation overrides; do not document authentication in prose only.
21. Use a single project-wide convention for correlation (`correlationId`) and trace propagation (W3C `traceparent` header) declared in `components.messageTraits` and applied to every operation; do not invent a different correlation field per message.
22. Mark payload and header fields explicitly as `required` when the producer guarantees them; do not rely on JSON Schema's "all properties optional by default" to communicate "all fields required."
23. Generate consumer and producer stubs, schemas, and human-readable docs from the AsyncAPI document in CI for every language the team supports; do not hand-maintain a parallel "should match the document" client.
24. Publish the AsyncAPI document to a discoverable location (an internal API catalog, a schema registry, or a versioned static-site URL) on every release; do not require consumers to clone the producing repo to find the contract.
