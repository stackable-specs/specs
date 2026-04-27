---
id: cdc
layer: practices
extends: []
---

# Change Data Capture

## Purpose

Change Data Capture (CDC) is an architectural pattern for propagating database mutations to downstream systems by tracking and publishing row-level changes as an ordered event stream. Without shared conventions, teams poll with timestamps that miss deletes, stream deltas without a baseline snapshot, publish mixed-table events that are impossible to replay correctly, and leave replication slots orphaned — producing inconsistent replicas, silent data loss, and storage-exhaustion incidents. This spec establishes the rules that keep CDC pipelines correct, recoverable, and operationally safe.

## Do

- Prefer log-based capture (WAL, binlog, change feeds) over query-based or trigger-based approaches.
- Perform a consistent initial snapshot before streaming incremental changes.
- Publish one dedicated topic or stream per source table.
- Design all consumers to be idempotent — CDC connectors deliver at least once.
- Monitor replication slot lag and alert before it threatens source storage.

## Don't

- Use timestamp-column polling as the primary capture method — it misses deletes and concurrent updates within the same poll window.
- Stream incremental changes without first establishing a full baseline snapshot.
- Multiplex changes from multiple source tables onto a single undifferentiated stream.
- Treat CDC as a synchronous request/response channel — it is an asynchronous, eventually consistent feed.
- Leave replication slots or log subscriptions open after a consumer is decommissioned.

## References

- **spec** `kafka` — common transport for CDC event streams
- **spec** `postgres` — log-based CDC via PostgreSQL logical replication (WAL)
- **external** `https://en.wikipedia.org/wiki/Change_data_capture` — CDC pattern overview and capture methods

## Rules

1. Use log-based capture (WAL for PostgreSQL, binlog for MySQL, change feeds for MongoDB/CosmosDB) as the preferred method; use query-based polling only when log access is unavailable, and account explicitly for missed deletes.
2. Perform a consistent, transaction-level initial snapshot of each source table before streaming incremental change events; do not stream deltas against a baseline that was never captured.
3. Include in every change event: the operation type (INSERT, UPDATE, DELETE), the full before-image and after-image of the row, and a monotonically increasing position identifier (LSN, binlog offset, or sequence number).
4. Publish change events for each source table to its own dedicated topic, stream, or channel; do not combine events from multiple tables in a single stream.
5. Design all CDC consumers to be idempotent with respect to duplicate events; do not assume exactly-once delivery from the capture layer.
6. Use the position identifier (LSN, offset) in every event to detect and discard duplicates at the consumer, rather than relying on wall-clock timestamps.
7. Never use a CDC stream as a synchronous API; consumers must tolerate lag between source commit and event availability.
8. Monitor replication slot lag (PostgreSQL), binary log retention, or equivalent source-side backpressure metrics; alert before unprocessed lag threatens source storage capacity.
9. Drop or release replication slots, log subscriptions, and consumer group offsets when a consumer is decommissioned; never leave orphaned slots open against a live source.
10. Version the change event schema and enforce backward-compatible evolution (add optional fields, never remove or rename required fields) so that source schema changes do not silently break consumers.
11. Store the last-processed position identifier in durable, transactional storage on the consumer side so that a restart resumes from the correct offset without re-processing the full history.
