---
id: postgres
layer: platform
extends: []
---

# PostgreSQL

## Purpose

Postgres is a multi-decade datastore: schema and convention choices made in week one are still binding when the table has billions of rows, the team has tripled, and zero-downtime deploys are non-negotiable. Most of the painful failure modes — `timestamp` columns that silently drop time zones, `NOT NULL` added to a hot table in a single migration, `varchar(255)` that nobody remembers the reason for, orphan rows from skipped foreign keys, application code connecting as `postgres` superuser, ad-hoc DDL from a developer's `psql` session — are not bugs in Postgres but the result of conventions the team did not enforce early. This spec pins schema, migration, query, and connection rules so the database stays evolvable under load, deploys remain zero-downtime, and "we'll clean it up later" does not become a multi-week migration project.

## References

- **external** `https://www.postgresql.org/` — PostgreSQL project home
- **external** `https://www.postgresql.org/docs/current/` — PostgreSQL documentation
- **external** `https://www.postgresql.org/docs/current/ddl.html` — DDL reference
- **external** `https://wiki.postgresql.org/wiki/Don%27t_Do_This` — PostgreSQL "Don't Do This" wiki
- **external** `https://use-the-index-luke.com/` — Use the Index, Luke (SQL indexing guide)

## Rules

1. Pin the Postgres major version per project (e.g. `postgres:17`) and document the upgrade cadence in the project README.
2. Run production on a managed Postgres service or a documented HA topology (primary + replica + automated backups); do not run production on a single self-hosted instance without a documented operational plan.
3. Require TLS on every client connection in non-local deployments; do not allow plaintext connections from outside `localhost`.
4. Take automated, restorable backups on a schedule that meets a documented RPO, and verify a restore at least quarterly.
5. Apply all DDL through a migrations tool (Flyway, sqitch, golang-migrate, Alembic, Prisma Migrate, etc.); do not run DDL ad hoc in production.
6. Add one forward migration per schema change; do not edit a migration that has been merged.
7. Use `timestamptz` for every time column; do not use plain `timestamp` (without time zone).
8. Use `text` for string columns by default; reach for `varchar(N)` only when `N` is a meaningful, enforced length constraint.
9. Use the `boolean` type for true/false fields; do not encode booleans as integers or strings.
10. Define foreign keys with explicit `ON DELETE` and `ON UPDATE` actions; do not rely on application code to maintain referential integrity.
11. Add `created_at` and `updated_at` `timestamptz` columns to every mutable table, with a trigger or framework hook maintaining `updated_at`.
12. Enforce enum-like value constraints with a `CHECK` constraint or a domain type; do not rely solely on application-side validation.
13. Make every migration backwards-compatible with the previously deployed application version so deploys are zero-downtime.
14. Add a `NOT NULL` column in multiple migrations (add nullable → backfill → set `NOT NULL`); do not combine the column add and the `NOT NULL` constraint in a single migration on a non-trivial table.
15. Rename or drop a column over a deprecation period during which both old and new columns coexist; do not drop a column in the same release that stops writing to it.
16. Add an index for every column used in a `WHERE`, `JOIN`, or `ORDER BY` clause on production traffic that is not already covered; do not add indexes speculatively to columns no query uses.
17. Use parameterized queries or prepared statements at every layer; do not interpolate user input into SQL strings.
18. Run `EXPLAIN ANALYZE` on any new query expected to scan more than ~10,000 rows or run on a hot path, and record the plan in the pull request.
19. Set a `statement_timeout` on every application database role to bound runaway queries.
20. Wrap multi-statement writes in explicit transactions, and choose an isolation level explicitly when stronger than the default `READ COMMITTED` is required.
21. Connect to production through a connection pooler (PgBouncer, RDS Proxy, or an in-process pool) sized to the database's connection limit; do not open a new connection per request.
22. Connect application code through a service-specific role with least-privilege grants; do not connect as the `postgres` superuser from application code.
23. Source database credentials from a secret manager and rotate them on a documented schedule; do not commit credentials to the repo or bake them into images.
24. Enable `pg_stat_statements` and forward query metrics and Postgres logs to the central observability store.
