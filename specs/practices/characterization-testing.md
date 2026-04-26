---
id: characterization-testing
layer: practices
extends: []
---

# Characterization Testing

## Purpose

A characterization test pins what code *currently does* — not what it *should do* — by running the code with a fixed input, capturing the observed output, and asserting that future runs produce the same output. Michael Feathers introduced the technique in *Working Effectively with Legacy Code* as the safety net that lets a team refactor unfamiliar code without specifying its intent first: lock the observable behavior, change the implementation, watch the locks. The technique is only valuable while it is recognized for what it is — a transitional scaffold, not a specification. The moment teams forget that distinction, characterization tests become permanent regression assets that lock in bugs ("the legacy code returns `None` here so the test asserts `None`"), accumulate alongside intentional tests with no way to tell them apart, get rubber-stamp regenerated whenever they fail (defeating their purpose), or get written for new code as a shortcut around designing real assertions. This spec pins when characterization tests are appropriate, how they are written and tagged, what their update workflow is, and the explicit retirement path that replaces each characterization with intentional unit / integration / property tests once the behavior is understood — so the technique stays a tool for retiring legacy unknowns rather than a category of unaccountable assertions.

## References

- **spec** `unit-testing` — primary tier characterization tests are eventually replaced by
- **spec** `integration-testing` — tier characterization tests use when the legacy boundary is I/O-heavy
- **spec** `mutation-testing` — assertion-strength check that exposes characterization tests asserting on the wrong values
- **spec** `tdd` — discipline used for the *replacement* tests that retire characterization tests
- **external** `https://en.wikipedia.org/wiki/Characterization_test` — Characterization test (Wikipedia)
- **external** `https://www.oreilly.com/library/view/working-effectively-with/0131177052/` — Michael Feathers, *Working Effectively with Legacy Code* (the original technique)
- **external** `https://approvaltests.com/` — ApprovalTests (cross-language characterization / approval framework)
- **external** `https://jestjs.io/docs/snapshot-testing` — Jest snapshot tests (a UI-focused subset of characterization testing)

## Rules

1. Write characterization tests only against existing code whose intended behavior is unknown, undocumented, or contested; do not write characterization tests for new code or code whose specification the team controls.
2. State the goal of every characterization test in a single comment at the top of the test (or its `describe` block): which behavior is being pinned, and what change the test is meant to enable; do not commit a characterization test with no recorded purpose.
3. Tag every characterization test so it is distinguishable from intentional spec-tests in tooling and reports (e.g. `@pytest.mark.characterization`, JUnit `@Tag("characterization")`, Jest `describe('[characterization] …')`); do not let characterization tests sit in the same bucket as unit tests under a single label.
4. Use an approval / golden-master framework appropriate to the language (`ApprovalTests`, `pytest-approvaltests`, Jest snapshots, JVM Approvals) that produces a versioned `received` / `approved` artifact pair; do not hand-roll string-equality assertions for outputs longer than a couple of lines.
5. Commit the captured `approved` (or snapshot) file alongside the test source in version control; do not regenerate the artifact in CI on demand.
6. Treat any divergence between `received` and `approved` as a failure that requires a human decision; do not configure CI to auto-update the `approved` artifact.
7. When the divergence reflects an intentional behavior change, regenerate the `approved` artifact in the same pull request that introduces the change, and reference the change record (BDR / ADR / ticket) in the commit message; do not regenerate without a written reason.
8. When the divergence reflects an unintentional change, investigate the regression rather than regenerating; do not "fix" a characterization test by accepting the new output.
9. Examine the `approved` artifact for behavior the team believes is wrong (returns `None` where `Error` was meant, silently truncates input, drops a required field) before committing it; do not freeze a known bug into the suite by approving its current output.
10. Replace each characterization test with an intentional unit / integration / property test as soon as the behavior it pins has been deliberately specified; do not maintain a characterization test alongside the intentional test that supersedes it.
11. Track open characterization tests as transitional debt — at minimum a count surfaced on the team's quality dashboard, ideally a list with an owner per area — and trend the count downward over time; do not treat characterization-test count as a metric to grow.
12. Run the legacy code under a deterministic environment (frozen clock, seeded RNG, fixed time zone, fixed locale, stubbed UUIDs) so the captured output is reproducible across machines and CI runs; do not approve an artifact that contains time, randomness, machine identifiers, or absolute paths.
13. Scrub captured output of secrets, internal URLs, PII, and machine-specific data before approving it; do not commit an `approved` artifact that contains credentials or PII even when the legacy code emits them.
14. When the legacy code is non-trivially I/O-bound (database, broker, HTTP), run the characterization under the project's integration-testing harness (per `specs/quality/integration-testing.md`) rather than mocking the I/O; do not characterize behavior at the wrong boundary.
15. When the captured output is large or binary, store it as a referenced fixture file (`test_<name>.approved.txt`, `test_<name>.approved.json`) next to the test; do not embed a thousand-line expected blob inline in the test source.
