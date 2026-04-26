---
id: testcontainers
layer: quality
extends:
  - integration-testing
---

# Testcontainers

## Purpose

Testcontainers is the library that turns "integration-test against the real dependency" from an aspiration into a default: a Postgres, a NATS, a Kafka, a localstack — each booted as a throwaway container per test run, wired to the application via a discovered host/port, and torn down at the end. The same library, used carelessly, becomes the slowest and flakiest part of CI: an unpinned `postgres` image rebases overnight and breaks the suite on a schema-incompatible point release, a shared module-scoped container leaks state between tests in unpredictable orderings, missing `wait_for` strategies let tests hit the broker before it has bound a port, the Ryuk reaper is disabled "for speed" and orphaned containers fill the runner's disk, network-bound HTTP fixtures hard-code `localhost:5432` instead of asking the container for its mapped port, and developers without Docker locally can't run the suite at all. This spec pins the testcontainers usage shape — pinned image references, lifecycle scoping, wait strategies, port discovery, network and volume hygiene, the relationship to the project's Docker / Compose specs, and the runtime/CI prerequisites — so integration tests stay fast, deterministic, and portable instead of degenerating into a docker-compose script masquerading as a test.

## References

- **spec** `integration-testing` — testing-discipline spec this operationalizes
- **spec** `docker` — image format and pinning rules testcontainers must honor
- **spec** `docker-compose` — alternative orchestration; testcontainers is the per-test version
- **external** `https://testcontainers.com/` — Testcontainers project home
- **external** `https://java.testcontainers.org/` — Testcontainers for Java
- **external** `https://testcontainers-python.readthedocs.io/` — Testcontainers for Python
- **external** `https://node.testcontainers.org/` — Testcontainers for Node.js
- **external** `https://golang.testcontainers.org/` — Testcontainers for Go
- **external** `https://java.testcontainers.org/features/wait_strategies/` — Wait strategies
- **external** `https://github.com/testcontainers/moby-ryuk` — Ryuk container reaper

## Rules

1. Use the language's official testcontainers binding (Java, Python, Node, Go, .NET, Rust, …) for every external dependency the integration suite needs (database, broker, object store, search engine); do not hand-roll `subprocess.run(["docker", ...])` calls in test setup.
2. Prefer the testcontainers-published module (`PostgreSQLContainer`, `KafkaContainer`, `LocalStackContainer`, …) over a generic `GenericContainer` when one exists; do not reimplement wait strategies or default ports a published module already encodes.
3. Reference every container image by digest (`@sha256:…`) or a specific immutable tag (`postgres:16.4-alpine`); do not start a testcontainers container from a floating tag (`postgres`, `latest`, `kafka:latest`) and do not let testcontainers' default tag pick the version.
4. Pin the testcontainers library version in the project's lockfile and pin the Ryuk reaper image referenced by the binding; do not allow either to float across CI runs.
5. Scope every container's lifecycle to the smallest unit the test needs — function for full isolation, module/session only when boot cost dominates and tests can prove they leave no shared state — and document the chosen scope in the fixture; do not default to a global session-scoped container without a reason.
6. Reset shared-scope state between tests deterministically (transactional rollback for SQL, namespace per test for Kafka topics / NATS subjects, fresh bucket per test for object stores); do not rely on test ordering to keep shared-scope state clean.
7. Set an explicit `wait_for` (or `wait_strategy`) on every container — log message, port listening, HTTP health endpoint, or a custom readiness probe — sized to the slowest plausible cold start; do not assume `start()` returning means the dependency is ready.
8. Read the host and mapped port from the container instance (`get_host()`, `get_exposed_port(...)`) at the moment the test connects; do not hard-code `localhost:<port>` in connection strings.
9. Set explicit per-container startup and per-test execution timeouts; do not let a hung container block the CI job until the runner-level timeout fires.
10. Leave the Ryuk reaper enabled (the testcontainers default); do not set `TESTCONTAINERS_RYUK_DISABLED=true` in CI without a documented exception covering how orphaned containers are cleaned up.
11. Mount only ephemeral, test-owned volumes; do not bind-mount source-tree paths or host-cache directories into a testcontainers container.
12. Forbid network egress to any host that is not another testcontainers container in the same test run, an in-process mock, or an explicitly allow-listed test domain; do not let a testcontainers test reach a real third-party API by accident.
13. Run integration tests that use testcontainers in a separate test target / marker (`@pytest.mark.integration`, JUnit `@Tag("integration")`, Go build tag, etc.) so contributors can run unit tests without Docker; do not interleave testcontainers fixtures into the unit-test path.
14. Require a Docker-compatible runtime (Docker, Podman, Colima, Rancher Desktop, Testcontainers Cloud) on the CI runner and document the supported set in the repository; do not assume the contributor's local laptop has Docker without an onboarding check.
15. Configure testcontainers' image pull policy and registry settings to use the project's allow-listed registry / mirror (per the `dependency-management` registry rules); do not let CI runs pull testcontainers images from arbitrary public registries when an internal mirror exists.
16. When the same dependency topology is exercised both by `docker-compose` (local dev) and by testcontainers (per-test), pin both to the same image digest so the two paths cannot diverge silently.
