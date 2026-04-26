---
id: nats
layer: platform
extends: []
---

# NATS

## Purpose

NATS is fast pub/sub and request-reply by default, with persistent streams and durable consumers via JetStream. The speed is only useful when the surface around it is shaped well: a meaningful subject taxonomy, typed message contracts, streams with explicit retention, consumers with explicit ack semantics, and W3C trace context that survives across hops. The evaluation defaults — no auth, no TLS, ephemeral consumers, untyped JSON, default retention, infinite reconnect — are fine for prototypes but unsafe in production: anyone on the network can publish, schemas drift between services, streams grow unbounded until disks fill, ack-less consumers lose work on restart, and traces stop at the wire. This spec pins NATS's hosting, subject, schema, JetStream, and client conventions so a team running NATS in production gets durable, traceable, authenticated messaging rather than a fast-but-fragile bus.

## References

- **spec** `openobserve` — observability target for NATS server logs and metrics
- **external** `https://nats.io/` — NATS project home
- **external** `https://docs.nats.io/` — NATS documentation
- **external** `https://docs.nats.io/nats-concepts/subjects` — Subject naming and wildcards
- **external** `https://docs.nats.io/nats-concepts/jetstream` — JetStream streams and consumers
- **external** `https://docs.nats.io/running-a-nats-service/configuration/securing_nats` — NATS security configuration
- **external** `https://www.w3.org/TR/trace-context/` — W3C Trace Context

## Rules

1. Pin the NATS server version via Docker image tag (`nats:<X.Y.Z>`) or signed binary release; do not deploy from `latest`.
2. Run NATS as a cluster of three or more server nodes for production; do not run production on a single-node deployment.
3. Provision persistent storage on every server node that hosts a JetStream stream replica.
4. Require TLS on client connections and inter-server routes in non-local deployments; do not accept plaintext NATS connections from outside `localhost`.
5. Use NATS accounts and decentralized auth (NKEYS, JWTs, or `nsc`-managed credentials) on every non-local deployment; do not run a NATS server with the default no-auth configuration.
6. Source NATS user credentials from a secret manager and rotate them on a documented schedule; do not commit credential files to the repo or bake them into images.
7. Use hierarchical, lowercase, dot-separated subject names of the form `<domain>.<entity>.<action>` (for example `orders.order.created`).
8. Reserve the `*` and `>` wildcards for subscribers; do not publish to subjects that contain wildcards.
9. Document the subject taxonomy in version control alongside the application code.
10. Define a message payload schema (CloudEvents, Protocol Buffers, Avro, or a committed JSON Schema) for every published subject; do not publish untyped JSON to a production subject.
11. Propagate the W3C `traceparent` (and `tracestate` when present) header in NATS message headers on every published message.
12. Set per-publisher and per-stream maximum message size limits explicitly; do not rely on the server default for production traffic.
13. Define every JetStream stream and consumer declaratively as committed configuration (NACK, NATS Operator, or `nats stream/consumer create` config files); do not configure streams interactively in production.
14. Set retention (`limits`, `interest`, or `workqueue`), max age, max bytes, max messages, and max message size explicitly on every stream.
15. Use durable consumers with explicit names for production workloads; do not use ephemeral consumers to process production messages.
16. Use queue groups (or shared durable consumers) when more than one instance must process the same subject; do not rely on independent subscriptions for load balancing.
17. Issue NATS request / reply calls with an explicit timeout; do not send requests without a deadline.
18. Acknowledge JetStream messages explicitly with `Ack`, `Nak`, or `Term`; do not enable auto-ack for production consumers.
19. Implement consumer handlers as idempotent operations; assume any message can be delivered more than once.
20. Set the consumer's delivery policy (`DeliverNew`, `DeliverAll`, `DeliverByStartTime`, `DeliverByStartSequence`) explicitly; do not rely on the default.
21. Configure clients to reconnect with bounded exponential backoff and a maximum retry count or duration; do not run with infinite immediate-retry on connection loss.
22. Drain client connections on graceful shutdown so in-flight messages complete before the process exits.
23. Enable the NATS monitoring endpoint, scrape its metrics into the team's metrics store, and forward NATS server logs to the central log store.
