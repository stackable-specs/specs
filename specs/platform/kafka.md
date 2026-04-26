---
id: kafka
layer: platform
extends: []
---

# Apache Kafka

## Purpose

Apache Kafka is a distributed, partitioned, replicated commit log that can serve as a durable event bus, a stream-processing substrate, and an integration backbone — but its defaults are calibrated for a developer laptop, not a production cluster: a single-broker `replication.factor=1` topic loses data the moment its leader dies, a producer running with the historical `acks=1` default acknowledges before followers replicate, an idempotent producer is opt-in, the consumer auto-commit timer can advance offsets past messages whose handlers crashed, retention defaults to seven days regardless of business meaning, an unkeyed producer scatters related events across partitions and destroys per-key ordering, the legacy ZooKeeper control plane has been removed in Kafka 4.0 in favor of KRaft, and an unauthenticated `PLAINTEXT` listener happily accepts writes from anyone on the network. This spec pins the cluster topology (KRaft, broker count, rack awareness), the topic contract (partitions, replication factor, min in-sync replicas, retention, cleanup policy), producer reliability (idempotence, `acks=all`, keys, compression, transactions where used), consumer correctness (group ids, manual commits, isolation level, rebalance protocol), schema governance (a registry with a compatibility mode), security (TLS, SASL, ACLs), and the operational baseline (JMX metrics, logs, declarative topology) so a Kafka deployment behaves like a durable event log rather than a fast-but-fragile message pipe.

## References

- **spec** `openobserve` — observability target for Kafka broker logs and JMX metrics
- **external** `https://kafka.apache.org/` — Apache Kafka home
- **external** `https://kafka.apache.org/documentation/` — Apache Kafka documentation
- **external** `https://kafka.apache.org/documentation/#kraft` — KRaft mode (ZooKeeper-less control plane)
- **external** `https://kafka.apache.org/documentation/#producerconfigs` — Producer configuration reference
- **external** `https://kafka.apache.org/documentation/#consumerconfigs` — Consumer configuration reference
- **external** `https://kafka.apache.org/documentation/#topicconfigs` — Topic configuration reference
- **external** `https://kafka.apache.org/documentation/#design_ha` — Replication and in-sync replicas
- **external** `https://kafka.apache.org/documentation/#design_compactionbasics` — Log compaction
- **external** `https://kafka.apache.org/documentation/#security` — Security (TLS, SASL, ACLs)
- **external** `https://docs.confluent.io/platform/current/schema-registry/index.html` — Confluent Schema Registry
- **external** `https://www.w3.org/TR/trace-context/` — W3C Trace Context

## Rules

1. Pin the Kafka broker version via Docker image tag (`apache/kafka:<X.Y.Z>` or equivalent vendor tag) or signed package release; do not deploy from `latest`.
2. Run Kafka in KRaft mode for any new cluster and migrate existing ZooKeeper-backed clusters before upgrading to Kafka 4.0; do not stand up a new ZooKeeper-backed Kafka cluster.
3. Run a production cluster with at least three brokers and at least three KRaft controller voters (combined or dedicated mode) on an odd quorum (3, 5, 7); do not run production on a single-broker or single-controller deployment.
4. Distribute brokers across at least three failure domains (availability zones or racks) and set `broker.rack` accordingly so partition replicas land on distinct racks.
5. Require TLS (`SSL` or `SASL_SSL`) on every client and inter-broker listener in non-local deployments; do not expose a `PLAINTEXT` listener outside `localhost`.
6. Authenticate every client connection with SASL (`SCRAM-SHA-512`, `OAUTHBEARER`, or mTLS) on non-local deployments; do not allow anonymous client connections to a production cluster.
7. Source broker keystores, truststores, and SASL credentials from a secret manager and rotate them on a documented schedule; do not commit secrets to the repo or bake them into images.
8. Enable broker authorization (`authorizer.class.name`) and grant per-principal ACLs scoped to the smallest set of topics, consumer groups, and transactional ids the principal needs; do not grant cluster-wide `ALL` to application principals.
9. Disable topic auto-creation (`auto.create.topics.enable=false`) and unclean leader election (`unclean.leader.election.enable=false`) on every production broker.
10. Declare topics, partition counts, replication factors, configs, ACLs, and quotas as committed configuration (Terraform, Strimzi `KafkaTopic`, or a checked-in `kafka-topics`/`kafka-configs` script); do not create or alter production topics interactively.
11. Create every production topic with `replication.factor >= 3` and `min.insync.replicas >= 2`; do not create a production topic with `replication.factor=1`.
12. Choose a partition count for each topic based on documented target throughput and consumer parallelism, and treat the chosen count as effectively immutable; do not increase partitions on a keyed topic without a documented re-keying plan because partition changes break per-key ordering.
13. Set `cleanup.policy` explicitly per topic — `delete` for event streams, `compact` for keyed state topics, `compact,delete` for hybrid — and set `retention.ms` (or `retention.bytes`) to a value justified by the business contract; do not rely on the broker default seven-day retention for a production topic.
14. Configure compacted topics with explicit `min.cleanable.dirty.ratio`, `segment.ms`, and `delete.retention.ms`; do not enable compaction on a topic whose keys are not stable, meaningful identifiers.
15. Define a message payload schema (Avro, Protocol Buffers, or JSON Schema) registered in a schema registry for every production topic, and configure the registry with a compatibility mode (`BACKWARD`, `FORWARD`, or `FULL`) appropriate to the topic's consumers; do not publish untyped JSON to a production topic.
16. Enable producer idempotence (`enable.idempotence=true`) and `acks=all` on every production producer; do not run a production producer with `acks=0` or `acks=1`.
17. Set producer `compression.type` explicitly (`zstd`, `lz4`, `snappy`, or `gzip`) at the producer or topic level; do not ship uncompressed production traffic.
18. Publish every record with a deterministic, business-meaningful key whenever per-key ordering or compaction is required; do not publish to a keyed topic with `null` keys.
19. Use Kafka transactions (`transactional.id`, `initTransactions`, `sendOffsetsToTransaction`) for any producer that must atomically write to multiple topics or commit consumer offsets alongside its writes; do not implement read-process-write exactly-once semantics with non-transactional producers.
20. Set producer `delivery.timeout.ms`, `request.timeout.ms`, `max.in.flight.requests.per.connection` (≤5 with idempotence), `linger.ms`, and `batch.size` explicitly per workload; do not run a production producer on the broker defaults.
21. Propagate the W3C `traceparent` (and `tracestate` when present) in Kafka record headers on every published record.
22. Assign every consumer a stable, application-scoped `group.id` per logical role and use the cooperative-sticky partition assignor (`partition.assignment.strategy=CooperativeStickyAssignor`); do not run production consumers with the eager range assignor.
23. Disable consumer auto-commit (`enable.auto.commit=false`) and commit offsets explicitly only after the handler has successfully processed the records; do not enable auto-commit for production consumers.
24. Set consumer `isolation.level=read_committed` for any consumer that reads from a topic written by a transactional producer.
25. Set consumer `max.poll.records`, `max.poll.interval.ms`, `session.timeout.ms`, `heartbeat.interval.ms`, and `fetch.min.bytes`/`fetch.max.wait.ms` explicitly per workload; do not run a production consumer on the broker defaults.
26. Implement consumer handlers as idempotent operations; assume any record can be redelivered after a consumer crash, rebalance, or commit failure.
27. Configure consumer error handling with a documented retry policy and a dead-letter topic for poison records; do not silently skip records or block a partition indefinitely on a single bad record.
28. Expose broker, producer, and consumer JMX metrics to the team's metrics store and forward Kafka broker logs to the central log store; do not operate a production cluster without metrics scraping.
