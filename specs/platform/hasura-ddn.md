---
id: hasura-ddn
layer: platform
extends: []
---

# Hasura DDN

## Purpose

Hasura DDN (Data Delivery Network) is a metadata-driven supergraph platform that generates a unified GraphQL API over multiple data sources. Without shared conventions, teams produce inconsistent metadata, couple subgraphs incorrectly, and unknowingly expose data by omitting permissions. This spec establishes the structural and operational rules that keep supergraph builds reproducible, subgraphs independently ownable, and the API surface governed.

## Do

- Define all API behavior in HML metadata files; let `ddn` generate engine build artifacts.
- Model each bounded domain as a separate subgraph with its own `subgraph.yaml`.
- Use `ddn connector introspect` to regenerate connector metadata after every schema change.
- Validate locally with `ddn supergraph build local` and Docker Compose before pushing cloud builds.
- Pin connector image versions in `connector.yaml` for reproducible builds.

## Don't

- Manually edit files under `engine/build/` — they are generated outputs.
- Deploy to any non-local environment with `NoAuth` mode in `AuthConfig`.
- Commit secrets or connection strings into HML files or `connector.yaml`.
- Omit permissions for any role — absence is an implicit deny, not a permissive default.

## References

- **external** `https://hasura.io/docs/3.0/getting-started/overview/` — Hasura DDN v3 getting started
- **external** `https://hasura.io/docs/3.0/supergraph-modeling/overview/` — Supergraph modeling and metadata reference
- **external** `https://hasura.io/docs/3.0/auth/overview/` — AuthConfig, JWT, and webhook authentication

## Rules

1. Author all API definitions as HML metadata files (`.hml`); never hand-edit generated JSON under `engine/build/`. (refs: https://hasura.io/docs/3.0/getting-started/overview/)
2. Organize each bounded domain as a separate subgraph with its own `subgraph.yaml`; the subgraph is the unit of team ownership and independent deployment.
3. Run `ddn connector introspect` to regenerate `DataConnectorLink` metadata whenever the underlying data source schema changes — do not hand-author connector metadata.
4. Set `AuthConfig` to JWT or webhook mode in all non-local environments; `NoAuth` mode is permitted only in local Docker-based development.
5. Declare `TypePermissions`, `ModelPermissions`, and `CommandPermissions` for every role that accesses the supergraph — the absence of a permission entry is an implicit deny.
6. Run `ddn supergraph build local` and verify behavior with Docker Compose before creating a cloud build.
7. Supply all secrets and connection strings through environment variables; do not commit credentials or DSNs into HML files or `connector.yaml`.
8. Pin connector image versions in `connector.yaml`; floating tags such as `latest` break reproducibility across environments.
9. Place cross-subgraph relationship definitions in the subgraph that owns the source model, not in the subgraph that owns the relationship target.
