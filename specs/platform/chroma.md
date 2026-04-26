---
id: chroma
layer: platform
extends: []
---

# Chroma

## Purpose

Chroma is a vector store whose similarity results are only meaningful when every vector in a collection was produced by the same embedding model — yet Chroma itself does not enforce that, and the common failure modes (different model per writer, default in-memory persistence, default distance metric, default unauthenticated server, unbounded query results, full source documents stuffed into metadata) all "work" until they don't. The easy-to-stand-up evaluation setup quietly turns into a production system that loses data on restart, exposes a write-capable embedding API to the network, returns silently wrong answers when models are mixed, and carries no record of which model produced which vectors. This spec pins Chroma's hosting, collection-design, query, and embedding-model provenance conventions so similarity results stay meaningful, the store stays durable and authenticated, and the data carries the metadata needed to debug or migrate it.

## References

- **external** `https://github.com/chroma-core/chroma` — Chroma source repository
- **external** `https://www.trychroma.com/` — Chroma project home
- **external** `https://docs.trychroma.com/` — Chroma documentation
- **external** `https://docs.trychroma.com/deployment` — Chroma deployment guide

## Rules

1. Pin the Chroma server version via Docker image tag (`chromadb/chroma:<X.Y.Z>`) or exact pip package version; do not deploy from `latest`.
2. Run Chroma with persistent storage (a mounted volume or managed Chroma Cloud) in every non-local deployment; do not run production on the ephemeral / in-memory client.
3. Enable Chroma's server-mode authentication on every non-local deployment; do not expose the API unauthenticated to a network.
4. Terminate TLS in front of the Chroma HTTP API (via reverse proxy or load balancer); do not serve embedding queries over plain HTTP across a network.
5. Pin the Chroma client library version in application dependencies to match the deployed server's major and minor version.
6. Name collections using a `<scope>-<purpose>` convention (for example `support-faq`, `agent-memory-prod`) and document the convention in the application repository.
7. Record the embedding model name and version on the collection's metadata at creation time.
8. Use exactly one embedding model (and therefore one vector dimension) per collection; do not write vectors produced by a different model into an existing collection.
9. Set the distance metric (`cosine`, `l2`, or `ip`) explicitly when creating a collection; do not rely on the default without a recorded rationale.
10. Document each collection's metadata schema (field names, types, allowed values) in version control alongside the application code, and update the documentation whenever the schema changes.
11. Use stable, idempotent document IDs (UUIDs or deterministic content hashes); do not generate fresh IDs on each upsert of the same document.
12. Keep per-document metadata within a documented size budget; store full source documents in an external object store and reference them from metadata by key or URL.
13. Apply filters server-side via the `where` and `where_document` parameters; do not retrieve unfiltered results and filter in the consumer.
14. Cap every query with an explicit `n_results` (or `k`); do not request unbounded result sets.
15. Specify `include` on every query to fetch only the fields the consumer uses (`documents`, `embeddings`, `distances`, `metadatas`).
16. Set a client request timeout on every Chroma call; do not let queries hang without bound.
17. Compute embeddings upstream of Chroma for production traffic; do not rely on Chroma's in-process embedding functions in production paths.
18. Record the embedding model name and version on each written document's metadata so consumers can detect mixed-model results at query time.
19. When changing the embedding model, create a new (or version-tagged) collection and re-embed source documents into it; do not mix vectors from different model versions in the same collection.
20. Exercise embedding-model upgrades and metadata-schema changes against a staging or cloned collection before applying them to production.
