---
id: debezium
layer: platform
extends:
  - cdc
---

# Debezium

## Purpose

Debezium is the platform that implements the CDC practice for production: a set of Kafka Connect source connectors (and the Debezium Server / Debezium Engine variants for non-Kafka sinks) that read each source database's native change log — Postgres WAL, MySQL binlog, MongoDB oplog, SQL Server CDC tables — and emit ordered, schemaed change events. The CDC pattern only delivers what `practices/cdc.md` promises when the connector is configured with intent: a uniquely named replication slot per connector, a declared snapshot mode, a pre-created publication or table allowlist, durable Kafka-backed offset and schema-history stores, schemaed serialization through a registry, signal-table-driven incremental snapshots, a dead-letter queue, and JMX metrics piped to the observability stack. Left to defaults, Debezium auto-names replication slots that survive a connector deletion (slowly filling the source's WAL until the database stops accepting writes), defaults to a file-based offset store that loses progress on a worker restart, snapshots the entire source under a long blocking lock, ships JSON without schemas that drift on every column rename, runs as the `postgres` superuser because nobody narrowed the role, and stops the entire pipeline on a single poison record. This spec pins how Debezium is deployed, how the source database is prepared, how snapshots are taken, how slots and publications are owned, how events are serialized, how schema and offset history are stored, how transforms are applied, how the connector is observed, and how it is decommissioned — so a Debezium pipeline is a load-bearing piece of infrastructure rather than a "we set it up once and stopped looking" failure mode waiting to happen.

## Do

- Run connectors on Kafka Connect (or Debezium Server) with the connector image / plugin version pinned per environment.
- Connect each connector with a dedicated, least-privilege source-database user that has only the privileges Debezium documents.
- Declare a uniquely named replication slot and pre-created publication per connector; release them explicitly on decommission.
- Store offset and schema history in durable, replicated Kafka topics — never the file-based stores.
- Serialize change events with Avro or Protobuf through a schema registry; emit tombstones on delete for compacted topics.
- Use the signal table to trigger ad-hoc incremental snapshots; reserve full re-snapshots for documented incidents.
- Configure a dead-letter queue and JMX-to-Prometheus metrics before the connector handles real traffic.

## Don't

- Configure connectors via ad-hoc `curl` against the Connect REST API; treat connector config as code under version control.
- Run a connector as a database superuser, or share a single DB role across connectors and environments.
- Let Debezium auto-name replication slots or auto-create publications without committing the names to the connector config.
- Use `JSON` without a schema, or a schema-less converter, in production pipelines.
- Use `snapshot.mode = never` without a documented reason explaining how the consumer will tolerate the missing baseline.
- Bake critical business logic into Single Message Transform (SMT) pipelines that are hard to test outside the connector.
- Take a connector offline by deleting it without first releasing its replication slot and publication.

## References

- **spec** `cdc` — practices-layer pattern this spec implements
- **spec** `kafka` — sibling platform-layer spec; default transport for Debezium events
- **spec** `postgres` — sibling platform-layer spec; required `wal_level`, slot, and publication settings live there
- **external** `https://debezium.io/documentation/` — Debezium documentation
- **external** `https://debezium.io/documentation/reference/stable/connectors/postgresql.html` — Debezium PostgreSQL connector reference
- **external** `https://debezium.io/documentation/reference/stable/connectors/mysql.html` — Debezium MySQL connector reference
- **external** `https://debezium.io/documentation/reference/stable/transformations/event-flattening.html` — `ExtractNewRecordState` SMT
- **external** `https://debezium.io/documentation/reference/stable/transformations/outbox-event-router.html` — Outbox `EventRouter` SMT
- **external** `https://strimzi.io/docs/operators/latest/overview.html#concepts-resources-kafkaconnector-str` — Strimzi `KafkaConnector` CRD for declarative Connect config

## Rules

1. Run Debezium connectors on a Kafka Connect cluster (or Debezium Server / Debezium Engine for non-Kafka sinks); do not run a one-off `debezium/connect` container as the production capture path.
2. Pin the Debezium connector / image version exactly per environment (e.g. `debezium/connect:3.0.7.Final`); do not deploy from `:latest` or a floating tag.
3. Run different major Debezium versions in development, staging, and production only across an explicit, documented upgrade window; do not allow long-term major-version drift between environments.
4. Manage connector configuration declaratively — committed JSON / YAML applied via `kcctl`, the Strimzi `KafkaConnector` CRD, or an equivalent GitOps tool; do not configure connectors with ad-hoc `curl` calls against the Connect REST API.
5. Connect each connector with a dedicated source-database user that has only the privileges Debezium documents (e.g. Postgres `LOGIN` + `REPLICATION` + `SELECT` on the captured tables, MySQL `REPLICATION SLAVE` + `REPLICATION CLIENT` + `SELECT`); do not connect as the database superuser.
6. Use a separate database role per connector instance; do not share one role across multiple connectors or environments.
7. For the PostgreSQL connector, set `plugin.name = pgoutput` and pre-create a `PUBLICATION` listing the captured tables; do not rely on Debezium's automatic publication creation in production.
8. Declare a unique, environment-scoped `slot.name` per connector (e.g. `<env>_<service>_dbz_slot`); do not let Debezium auto-name the replication slot.
9. Configure `topic.prefix` (formerly `database.server.name`) per environment and per logical source so topics are namespaced (e.g. `prod_orders`); do not reuse the same `topic.prefix` across environments.
10. Set `snapshot.mode` explicitly per connector (`initial`, `initial_only`, `when_needed`, `no_data`, `recovery`, etc.); do not rely on the connector default. (refs: cdc)
11. Use `snapshot.mode = initial` or an incremental-snapshot strategy for new connectors; do not use `snapshot.mode = never` without a documented reason explaining how downstream consumers tolerate the missing baseline. (refs: cdc)
12. For tables larger than the connector can snapshot in the documented blocking-snapshot window, use Debezium's incremental snapshotting (`incremental.snapshot.chunk.size`, signal-table trigger) instead of a blocking initial snapshot.
13. Trigger ad-hoc re-snapshots through the signal table (e.g. `debezium_signal`) using `execute-snapshot` with `data-collections` listed; do not delete and recreate the connector to force a re-snapshot of one table.
14. Store connector offsets in a durable, replicated Kafka topic via `offset.storage = KafkaOffsetBackingStore` (Connect distributed mode); do not use `FileOffsetBackingStore` for production. (refs: cdc)
15. For the MySQL and SQL Server connectors, store schema history in a dedicated Kafka topic via `schema.history.internal.kafka.topic`; do not use `FileSchemaHistory` in production.
16. Set the offset, config, and schema-history Kafka topics to `cleanup.policy = compact`, `min.insync.replicas ≥ 2`, and a replication factor matching the broker's standard for durable topics.
17. Enable heartbeat events on low-volume sources via `heartbeat.interval.ms` so the replication slot's restart LSN advances and the source WAL does not accumulate; do not leave heartbeats disabled on a slot whose source has hours of no writes.
18. Set `tombstones.on.delete = true` (the default) when downstream Kafka topics use `cleanup.policy = compact`; do not disable tombstones on a compacted topic.
19. Serialize keys and values with Avro, Protobuf, or JSON Schema through a schema registry (Confluent Schema Registry, Karapace, or AWS Glue); do not run production pipelines with the schema-less JSON converter. (refs: cdc)
20. Configure the schema registry's compatibility mode to `BACKWARD` (or stricter) for every Debezium subject; do not allow incompatible schema evolutions to land. (refs: cdc)
21. Apply the `ExtractNewRecordState` SMT (formerly `UnwrapFromEnvelope`) when downstream consumers expect flat row records, and document — in the connector config or a sibling README — that the envelope's before-image and operation type are intentionally dropped for those consumers.
22. For application-emitted domain events, use the Outbox pattern with the `EventRouter` SMT against an outbox table the application writes transactionally; do not derive domain events from raw row-level change events for tables the application owns.
23. Configure a dead-letter queue with `errors.tolerance = all`, `errors.deadletterqueue.topic.name = <env>_<connector>_dlq`, and `errors.deadletterqueue.context.headers.enable = true`; do not let a single poison record stop the connector with `errors.tolerance = none`.
24. Expose Debezium and Kafka Connect JMX metrics through the Prometheus JMX exporter, and alert on `MilliSecondsBehindSource`, `NumberOfDisconnects`, `QueueRemainingCapacity`, and `LastEvent` staleness before a slot's WAL retention runs out.
25. Pause the connector (`PUT /connectors/<name>/pause`) before planned source-database maintenance, and resume after the source returns; do not let the connector observe a flapping source.
26. When retiring a connector, drop its replication slot and (for Postgres) drop or alter its `PUBLICATION` to remove the connector's tables in the same change; do not delete a Connect connector resource without releasing its source-side artifacts. (refs: cdc)
27. Restrict the Connect worker's network egress to the source database and the Kafka broker (and the schema registry where used); do not allow general outbound network access from the connector's host or pod.
28. Cover Debezium connectors in integration tests using the official `debezium/connect` container (or testcontainers `DebeziumContainer`) against a real source database; do not mock the connector with hand-built event fixtures.
29. Document the operational runbook for each connector — slot name, expected lag, snapshot strategy, signal-table location, DLQ topic, retire procedure — alongside its committed config; do not leave this knowledge to tribal memory.
