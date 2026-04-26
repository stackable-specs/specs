---
id: pgvector
layer: platform
extends:
  - postgres
---

# pgvector

## Purpose

pgvector turns Postgres into a viable vector store, but only when the column dimension, distance operator, index type, and embedding model are all aligned. The minimal-configuration path ("`CREATE EXTENSION vector`, add a `vector` column, start querying") works on toy datasets and silently degrades into linear scans, operator-class / query-operator mismatches, vectors from different models stored in the same column, IVFFlat indexes built before data is loaded, and similarity queries running with default `hnsw.ef_search` or `ivfflat.probes` — all of which produce plausible-looking but incorrect nearest-neighbor results without raising errors. This spec refines the `postgres` spec with vector-specific rules so similarity search in Postgres carries the same correctness and performance guarantees as the rest of the database.

## References

- **spec** `postgres` — base RDBMS rules this spec refines
- **spec** `chroma` — sibling vector-store option (separate service rather than a Postgres extension)
- **external** `https://github.com/pgvector/pgvector` — pgvector source repository
- **external** `https://github.com/pgvector/pgvector#indexing` — pgvector indexing guide
- **external** `https://github.com/pgvector/pgvector#performance` — pgvector performance guide

## Rules

1. Install pgvector via `CREATE EXTENSION vector` in a versioned migration; do not run `CREATE EXTENSION` ad hoc against a production database.
2. Pin the pgvector extension version with `CREATE EXTENSION vector VERSION '<X.Y.Z>'` and document the version per database; do not let the extension version float.
3. Declare every vector column with an explicit dimension (`vector(N)`); do not use `vector` without a dimension argument in committed schemas.
4. Set the column dimension to match the embedding model's output exactly; do not pad, truncate, or auto-coerce vectors of a different dimension into the column.
5. Use one embedding model per vector column; if a second model is introduced, store its vectors in a separate column or table.
6. Record the embedding model name and version on every row (in a sidecar column or join table) so mixed-model rows can be detected at query time.
7. Choose the distance operator deliberately (`<->` for L2, `<=>` for cosine, `<#>` for negative inner product) and document the choice in the schema documentation.
8. Index every vector column used in similarity queries with HNSW or IVFFlat before the table grows past roughly 10,000 rows; do not run `ORDER BY` similarity scans on un-indexed columns at scale.
9. Match the index's operator class to the query operator (`vector_l2_ops`, `vector_cosine_ops`, `vector_ip_ops`); do not pair a cosine query with an L2-indexed column.
10. Build IVFFlat indexes after the table is populated, with `lists` chosen per the pgvector tuning guide (approximately `sqrt(rows)` or `rows / 1000`); do not build IVFFlat with the default `lists` on a populated table.
11. Build HNSW indexes with `m` and `ef_construction` parameters chosen for the use case; do not rely on defaults without recording a tuning rationale.
12. Set `hnsw.ef_search` (HNSW) or `ivfflat.probes` (IVFFlat) per session or query to balance recall and latency; do not run production traffic with the server defaults without measuring recall.
13. Cap every similarity query with an explicit `ORDER BY <vector_col> <op> <query> LIMIT k`; do not return unbounded similarity result sets.
14. Pass query vectors as bound parameters (`$1::vector`) to similarity queries; do not interpolate vector literals into SQL strings.
15. Re-embed the column and rebuild its index after changing the embedding model; vectors from the previous model are not comparable to new ones via the same operator.
16. Exercise embedding-model upgrades and index-parameter changes against a staging dataset (with realistic row count and dimension) before applying them to production.
17. Use `halfvec` (pgvector 0.7+) or sparse vector types when the storage cost of full `vector(N)` is a constraint and recall targets allow; do not default to `float32` storage when a benchmarked smaller representation suffices.
