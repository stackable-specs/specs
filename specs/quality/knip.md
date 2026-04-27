---
id: knip
layer: quality
extends: []
---

# Knip

## Purpose

Knip finds the dead weight in a JavaScript / TypeScript project that linters and bundlers structurally cannot — unused files, unused exports and exported types, duplicate exports, unused class members, declared dependencies nothing imports, and *unlisted* dependencies the code imports without declaring. Linters operate file-by-file; Knip walks the whole module graph from the declared entry points and reports what nothing reaches. The discipline only pays off when entry points are declared explicitly, the graph is bounded by a `project` glob, framework plugins are wired so route handlers / build configs / test files / Storybook stories are not flagged as "unused," and CI fails on findings rather than printing a daily-growing list of debt that everyone scrolls past. Left to defaults, Knip silently treats a misconfigured entry as "everything is unused," contributors add `--no-config-hints --include-libs` flags to make warnings vanish, the team accumulates broad `ignore` arrays in `knip.json` instead of fixing the underlying dead code, and unlisted dependencies (the kind that work locally because of hoisting and break in production) are never reported because the report is too noisy to read. This spec pins how Knip is versioned, configured, scoped, plugged into frameworks, gated in CI, and suppressed when truly necessary so that "Knip clean" means the tree of reachable code is the tree the team intends to ship — and the dependency graph in `package.json` is the graph the code actually uses.

## Do

- Pin Knip exactly in `devDependencies` and run it through the project package manager.
- Declare `entry` patterns explicitly per workspace; let Knip's framework plugins auto-detect, but verify the result.
- Bound analysis with a `project` glob so generated, vendored, and out-of-source files are excluded from the start.
- Run `knip --strict` (or equivalent) in CI on every PR and treat any finding as a build failure.
- Pair Knip with Biome — Biome owns style and lint, Knip owns reachability and dependency hygiene.
- Suppress an individual finding inline at the violation site with a real reason; reserve `ignore` arrays for genuinely unfixable cases.
- Run Knip on a recurring schedule (post dependency updates) in addition to per-PR runs.

## Don't

- Run a globally-installed Knip from package scripts, hooks, or CI.
- Add broad `ignore`, `ignoreDependencies`, `ignoreBinaries` entries to silence findings instead of fixing the underlying dead code.
- Rely on a linter's "no unused vars" rule to catch module-level dead exports — it operates per file and cannot see cross-module reachability.
- Run `knip --fix` (or `--fix-type`) in CI; reserve auto-fixes for local invocations under PR review.
- Bump Knip silently in a routine dependency PR — major versions can surface a backlog of new findings.
- Treat an "unlisted dependency" as a warning to ignore; it is the supply-chain hole the policy exists to close.
- Use per-file pragma comments to silence findings on generated code; configure the `project` glob to exclude it.

## References

- **spec** `biome` — sibling quality-layer spec that owns style and lint for the same source tree
- **spec** `dependency-management` — security-layer spec; Knip implements its rule 16 (unused-dependency removal) for JS / TS
- **spec** `prek` — sibling quality-layer spec for the hook runner that invokes Knip locally
- **external** `https://knip.dev/` — Knip home
- **external** `https://knip.dev/overview/configuration` — Knip configuration reference
- **external** `https://knip.dev/reference/cli` — Knip CLI reference
- **external** `https://knip.dev/reference/issue-types` — Knip issue type catalogue
- **external** `https://knip.dev/reference/plugins` — Knip plugin catalogue (React, Next, Vite, Astro, Storybook, …)
- **external** `https://knip.dev/features/monorepos` — Knip monorepo / workspaces guidance

## Rules

1. Adopt Knip as the canonical reachability and dependency-hygiene analyzer for every JavaScript and TypeScript project; do not maintain a parallel home-grown script for unused-export or unused-dependency detection.
2. Pin the `knip` version as an exact `devDependency` in `package.json` (no `^`, `~`, or `>=`); do not depend on a floating range.
3. Configure Knip declaratively in a committed `knip.json`, `knip.jsonc`, `knip.ts`, or `knip.config.ts` at the repository root (or per workspace in a monorepo); do not pass long CLI flag lists from `package.json` scripts.
4. Set the `$schema` field in `knip.json` to the schema URL matching the pinned Knip version (e.g. `https://unpkg.com/knip@<version>/schema.json`); do not leave `$schema` unset or pinned to a stale version.
5. Declare `entry` patterns explicitly for every workspace — application entry points, CLI bins, build / config files, test files, Storybook stories, framework conventions — and verify the captured set against `knip --reporter symbols`; do not rely on Knip's defaults to discover non-obvious entry points.
6. Bound the analyzed file set with a `project` glob (e.g. `["src/**/*.{ts,tsx}", "tests/**/*.ts"]`); do not let Knip walk `node_modules`, generated output, vendored bundles, or build caches.
7. Enable Knip's framework plugins for every framework present in the project (React, Next, Remix, Vite, Astro, SvelteKit, Storybook, Vitest, Jest, Playwright, etc.) and audit the plugin's auto-discovered entries on first adoption; do not silently disable a plugin to suppress findings.
8. Run `knip --strict` (or `knip --include unlisted,unresolved,binaries,exports,types,duplicates,enumMembers,classMembers,nsExports,nsTypes`) as a single CI gate on every pull request; treat any finding as a build failure.
9. Run Knip in CI in report-only mode (no `--fix`); do not let CI auto-apply Knip's removals.
10. Run Knip through the project package manager (`pnpm knip`, `npm exec knip`, `bunx knip`, `yarn knip`); do not invoke a globally-installed Knip from package scripts, hooks, or CI.
11. Use `knip --fix` (or `--fix-type <type>`) only in local invocations and pre-commit hooks under PR review; do not run `--fix` against generated code or vendored sources.
12. Treat every `unlisted` and `unresolved` dependency finding as a build failure; do not allow code to import a package that is not declared in `package.json`. (refs: dependency-management)
13. Treat every `unused dependency` finding as a build failure; remove the declaration in the same change and re-run Knip to confirm. (refs: dependency-management)
14. Suppress an individual finding inline at the violation site (`// knip-ignore <reason>` or the framework's documented pragma) with a real reason citing why the export / dependency is intentionally retained; do not silence findings by removing the issue type from the report.
15. Configure path-scoped exceptions through the `ignore`, `ignoreDependencies`, `ignoreBinaries`, `ignoreExportsUsedInFile`, or `paths` fields in `knip.json` only with an inline comment justifying each entry; do not maintain unannotated arrays of suppressions.
16. Exclude generated files from analysis by listing them in the `project` glob's negative patterns or in `ignore`; do not annotate generated files with per-file pragmas the codegen would overwrite.
17. For monorepos, configure `workspaces` so each package gets its own `entry` and `project` definition, and run Knip per workspace; do not rely on a single root configuration to cover heterogeneous packages.
18. For published packages, declare the public API surface in `entry` so legitimate public exports are not flagged as unused; do not silence the package's exports broadly with `ignoreExportsUsedInFile` or a wildcard.
19. Pair Knip with the project's linter (Biome, ESLint) — Knip owns reachability and dependency hygiene, the linter owns style and per-file lint; do not rely on the linter's "no unused vars" rule to catch module-level dead exports. (refs: biome)
20. Run Knip on a recurring schedule (e.g. nightly or weekly cron) in addition to per-PR runs so dependency-update PRs that turn previously-used exports into dead code are caught promptly.
21. Bump Knip via a dedicated pull request that reviews the new finding set and adjusts `entry` / `ignore` deliberately; do not bundle a Knip upgrade into an unrelated dependency PR.
22. Invoke Knip from the project's hook runner (`prek`, `lefthook`, `husky`, etc.) on the changed-files set for fast pre-commit feedback; do not require contributors to install hooks by hand. (refs: prek)
