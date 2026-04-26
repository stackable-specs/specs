---
id: test-coverage
layer: quality
extends: []
---

# Test Coverage Metrics

## Purpose

Test coverage is one of the easiest software metrics to measure and one of the easiest to misuse. Reported as a single repo-wide percentage, it reduces a noisy, multi-dimensional question — "which behavior is exercised by which test, and would the test catch a regression?" — to a number teams can pad with assertion-free tests, dilute by counting generated code, or hold flat for years while the actual at-risk surface grows. Reported well, the same metric is a real signal: line and branch coverage targeted by tier, gated on new code, scoped to first-party logic, paired with a ground-truth check (mutation testing, BDR scenarios) that confirms the executed lines were actually asserted on. Beyond code coverage, requirements coverage (every BDR has at least one scenario), risk coverage (every high-risk component has a targeted test), and test-execution coverage (every committed test ran on every gated build) close the gaps a code-coverage number cannot see by itself. This spec pins which coverage dimensions the team measures, how they are computed, where the thresholds sit, what is excluded and why, how decline is handled, and the rule that coverage is a floor on assertion presence — not a ceiling that excuses mutation, integration, or property tests from carrying their share of the suite.

## References

- **spec** `unit-testing` — primary tier whose code coverage this spec quantifies
- **spec** `integration-testing` — coverage tier with separate thresholds and an instrumented run
- **spec** `property-based-testing` — coverage that example-based tests miss
- **spec** `mutation-testing` — ground-truth check that asserted-on lines actually catch regressions
- **spec** `bdr` — source of requirements coverage (every BDR maps to ≥1 scenario)
- **external** `https://www.practitest.com/resource-center/blog/test-coverage-metrics/` — Test coverage metrics overview
- **external** `https://martinfowler.com/bliki/TestCoverage.html` — Martin Fowler on coverage as a tool, not a target
- **external** `https://en.wikipedia.org/wiki/Code_coverage` — Code coverage taxonomy (line, branch, function, condition, path)
- **external** `https://about.codecov.io/` — Codecov (coverage report aggregation)
- **external** `https://docs.sonarsource.com/sonarqube/latest/user-guide/metric-definitions/#coverage` — SonarQube coverage definitions
- **external** `https://github.com/marketplace/actions/coverage-report-action` — diff-coverage gating in CI

## Rules

1. Measure code coverage on every CI run that exercises the test suite; do not run tests without coverage instrumentation in a gated job.
2. Report at minimum line coverage **and** branch coverage; do not gate on line coverage alone — uncovered branches inside covered lines are precisely where defects hide.
3. Set explicit numeric thresholds per tier in committed configuration (`pyproject.toml` `[tool.coverage]`, `jest.config.js` `coverageThreshold`, `nyc.config.js`, `JaCoCo` rules, etc.); do not rely on a verbal team agreement or a per-PR judgment call.
4. Set a higher threshold on **new and changed code in the PR** than on the repository aggregate (e.g. `--diff-cover --fail-under=90` for new code while overall sits at 80); do not let a PR pass a flat global threshold while the changed surface is uncovered.
5. Block CI on any coverage drop below threshold; do not configure the coverage report as `continue-on-error` or as informational-only.
6. Block CI on any per-package or per-component coverage drop the project tracks, not only on the repo-wide aggregate; do not let a critical module's coverage decay while a well-tested module's coverage masks it in the global average.
7. Exclude only generated, vendored, third-party, or build-tooling source paths (e.g. `**/*_pb2.py`, `migrations/`, `node_modules/`, `dist/`); declare every exclusion in the coverage config with a one-line comment citing the reason.
8. Do not exclude first-party application code from coverage measurement to lift the percentage; if a module is genuinely untestable, refactor it or document the carve-out in a record (ADR / BDR) rather than hiding it from the metric.
9. Compute coverage from instrumented test runs across **every** test tier the project gates on (unit, integration, property), then merge the per-tier reports into a single coverage report; do not gate on a unit-test-only coverage number when integration tests cover code paths unit tests cannot.
10. Pair code-coverage gating with a periodic mutation-testing run (per `mutation-testing` spec) so executed-and-asserted-on is checked against executed-but-not-asserted-on; do not treat 100% line coverage as evidence the assertions are sufficient.
11. Track requirements coverage by linking every BDR (per `bdr` spec) to at least one executable scenario in the integration suite; do not allow a BDR to ship without a backing test.
12. Track risk coverage by maintaining a list of high-risk components (security boundaries, payment paths, data-migration code, public API surface) and requiring an explicit owner-attested test plan for each; do not rely on the global coverage percentage to defend a high-risk component.
13. Track test-execution coverage by failing CI when any committed test is unexpectedly skipped, marked `xfail`, or filtered out by a marker in a gated job (`pytest --strict-markers`, `jest --passWithNoTests=false`); do not let coverage rise because tests stopped running.
14. Forbid assertion-free or assertion-trivial tests (a test that calls a function but asserts only `is not None`, or a property test whose body is `pass`); enforce via a lint rule, an editor-time check, or a code-review checklist.
15. Publish coverage reports as committed CI artifacts in a standard format (`coverage.xml` Cobertura, `lcov.info`, JaCoCo XML, `coverage.json`) and upload them to the team's coverage store (Codecov, Coveralls, SonarQube, an S3 bucket); do not keep coverage data only in per-job CI logs.
16. Require the coverage delta (`+/-X%`, both overall and on changed files) to appear as a status check or a comment on every pull request; do not require reviewers to download an artifact to find out whether coverage moved.
17. Review the coverage thresholds at a documented cadence (quarterly minimum) and ratchet them upward when sustained coverage exceeds the floor; do not freeze thresholds at the value chosen at project inception.
18. Document the coverage policy (which metrics are measured, the per-tier thresholds, the exclusion list, the upload destination, the owner) in the repository or a linked policy doc; do not enforce coverage rules only as tribal knowledge.
