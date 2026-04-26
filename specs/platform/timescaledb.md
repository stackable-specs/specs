---
id: timescaledb
layer: platform
extends:
  - postgres
---

# TimescaleDB

## Purpose

TimescaleDB makes Postgres practical for high-volume time-series workloads through hypertables (automatic time-based partitioning), columnar compression of older chunks, continuous-aggregate materialized views, and declarative retention policies. The value only materializes when those features are configured deliberately. The minimal-config path — `CREATE EXTENSION timescaledb`, insert rows, query — works for demos and silently degrades into a single 100-million-row hypertable whose chunks are too large to fit in `shared_buffers`, whose disk usage doubles every quarter because nothing is compressed, whose retention is configured nowhere, and whose dashboards re-aggregate raw data on every page load. The result is a Postgres database that performs worse on time-series than vanilla Postgres would. This spec refines the `postgres` spec with the hypertable, compression, retention, continuous-aggregate, and query conventions that keep TimescaleDB performant under production load.

## References

- **spec** `postgres` — base RDBMS rules this spec refines
- **spec** `pgvector` — sibling Postgres-extension spec following the same `extends` pattern
- **external** `https://github.com/timescale/timescaledb` — TimescaleDB source repository
- **external** `https://docs.timescale.com/` — TimescaleDB documentation
- **external** `https://docs.timescale.com/use-timescale/latest/hypertables/` — hypertables guide
- **external** `https://docs.timescale.com/use-timescale/latest/compression/` — compression guide
- **external** `https://docs.timescale.com/use-timescale/latest/continuous-aggregates/` — continuous aggregates

## Rules

1. Install TimescaleDB via `CREATE EXTENSION timescaledb` in a versioned migration; do not run `CREATE EXTENSION` ad hoc against a production database.
2. Pin the TimescaleDB extension version with `CREATE EXTENSION timescaledb VERSION '<X.Y.Z>'` and document the version per database; do not let the extension version float.
3. Convert every time-indexed table that accepts large volumes of time-stamped writes into a hypertable via `create_hypertable()`; do not leave high-volume time-series tables as plain Postgres tables.
4. Choose `chunk_time_interval` explicitly based on the expected ingest rate, sized so that recent chunks fit in `shared_buffers`; do not accept the default chunk interval without a sizing rationale.
5. Use the table's primary time column as the hypertable partition key in `create_hypertable()`; do not partition by a non-time column without a documented reason.
6. Add explicit indexes on the columns most often used in `WHERE` and `JOIN` clauses on a hypertable; the default time-only index does not cover other access patterns.
7. Ingest in batches via `COPY` or batched `INSERT` statements; do not perform row-by-row inserts at high rates against a production hypertable.
8. Enable native compression on every hypertable whose data is historically queried more than written, and choose `segmentby` and `orderby` columns based on the dominant query pattern.
9. Apply compression automatically with `add_compression_policy(<table>, INTERVAL '<age>')`; do not rely on manual `compress_chunk()` calls in production.
10. Apply a retention policy via `add_retention_policy(<table>, INTERVAL '<age>')` so hypertables do not grow unbounded, and document the retention period for each hypertable in version control.
11. Use continuous aggregates (`CREATE MATERIALIZED VIEW ... WITH (timescaledb.continuous)`) for rollup queries that run on every dashboard load; do not recompute identical aggregations on raw data repeatedly.
12. Schedule continuous-aggregate refresh with `add_continuous_aggregate_policy(...)` specifying `start_offset`, `end_offset`, and `schedule_interval` explicitly; do not rely on real-time aggregation alone for high-volume views.
13. Set `materialized_only` deliberately on every continuous aggregate to declare whether real-time (raw + materialized) results are required.
14. Use TimescaleDB time-series functions (`time_bucket`, `time_bucket_gapfill`, `first`, `last`, `locf`, `interpolate`) for bucketed and gap-filled queries; do not reimplement bucketing with `date_trunc` when a Timescale function applies.
15. Filter queries by the hypertable's time column first so chunk-exclusion can prune chunks; do not run queries on production traffic that necessarily scan every chunk.
16. Monitor TimescaleDB background-job health via `timescaledb_information.job_stats`, forward the results to the team's metrics store, and alert on stalled compression, retention, or refresh jobs.
17. Configure `timescaledb.max_background_workers` for the expected concurrent policies (compression + retention + continuous-aggregate refresh) and review the setting whenever a new policy is added.
