---
id: mutation-testing
layer: quality
extends: []
---

# Mutation Testing

## Purpose

A passing test suite only tells you that the tests did not fail; it does not tell you the tests would catch a regression. Line and branch coverage tell you which code was executed; they do not tell you whether the assertions on that code actually check anything. Mutation testing closes that gap: introduce a small syntactic change to the source (flip `+` to `-`, change `<` to `<=`, drop a `return`, swap a boolean literal) and re-run the tests. A mutant the suite kills means at least one test would have caught a regression of that shape; a mutant that survives reveals a hole — somewhere a real bug in that shape would also slip through unnoticed. The discipline only pays off when scores are gated in CI, surviving mutants are investigated rather than bulk-ignored, and runs target pure logic where mutants represent real defects rather than infrastructure noise. This spec pins how mutation testing is configured, scoped, scored, and integrated so test-suite quality becomes a measurable property the team gates on, not a gut-feel judgment based on coverage percentages.

## References

- **spec** `property-based-testing` — sibling test-methodology spec for the `quality` layer
- **spec** `tdd` — sibling testing-discipline spec for the `practices` layer
- **external** `https://en.wikipedia.org/wiki/Mutation_testing` — Mutation testing overview
- **external** `https://stryker-mutator.io/` — Stryker (JS / TS / .NET / Scala mutation framework)
- **external** `https://pitest.org/` — PIT / Pitest (Java / Kotlin mutation framework)
- **external** `https://github.com/boxed/mutmut` — mutmut (Python mutation framework)

## Rules

1. Use a dedicated mutation-testing framework appropriate to the language: Stryker (JS / TS / .NET / Scala), PIT (Java / Kotlin), mutmut or Cosmic Ray (Python), go-mutesting (Go), mutator-rs (Rust); do not roll your own mutation logic.
2. Pin the mutation-testing framework version in build configuration; upgrade deliberately via a dedicated PR.
3. Apply mutation testing to pure-logic modules (parsers, validators, math, codecs, business rules); do not run mutation testing across modules dominated by I/O or framework wiring where most mutants would be infrastructure noise.
4. Run incremental mutation testing on changed code in pull-request CI; do not run full-suite mutation testing on every PR when the run is slow enough to block PR throughput.
5. Run full-suite mutation testing on a scheduled cadence (nightly or weekly) and route a drop in the overall score to the team's alerting channel.
6. Set a minimum mutation score per module (commonly 70–80% for new modules, with a documented baseline for legacy code); do not run mutation testing without a configured threshold.
7. Fail pull-request CI when the mutation score for changed files falls below the configured threshold.
8. Investigate every survived mutant: either add a test that kills it, mark the mutator as ignored at the call site with a comment, or record it as an equivalent or infeasible mutant in a tracked file.
9. Do not bulk-suppress survived mutants; each suppression entry must cite a reason (equivalent mutant, intentional behavioral choice, infeasible test).
10. Keep an "ignored mutants" file (or framework equivalent) in version control and review it during code review of test changes.
11. Assert on observable behavior — return values, persisted state, externally visible output — not on internal log lines; mutants that only change log text do not represent real defects and waste investigation budget.
12. Write multiple targeted assertions per behavior so a mutant that breaks one aspect of the output is caught even if another assertion still passes.
13. Enable the framework's default mutator set; do not disable a mutator category without documenting why it produces only equivalent or infeasible mutants in this codebase.
14. Cache previous mutation runs (Stryker `incremental`, PIT `--withHistory`, mutmut's cache, or framework equivalent) so unchanged code is not re-mutated; do not re-mutate unchanged files in CI.
15. Treat mutation-testing failures as quality-gate failures, not advisory warnings.
16. When a survived mutant reveals a real test gap, write a failing test that the mutant kills before adjusting other code; verify the new test kills the mutant.
