---
id: biome
layer: quality
extends: []
---

# Biome

## Purpose

Biome is a single Rust-implemented binary that lints, formats, and organizes imports for JavaScript, TypeScript, JSX, TSX, JSON, JSONC, and CSS — replacing the historical ESLint + Prettier + an import-sort plugin stack with one tool, one config file, one cache, and one CI gate. The wins (speed, fewer config files, deterministic formatting) only land when the team commits fully: running Biome alongside ESLint or Prettier reintroduces every drift Biome exists to remove (two formatters fighting over quote style, two linters disagreeing about an unused-import rule, three caches to invalidate). Left to defaults, Biome happily formats whatever a contributor configured locally, ignores files outside the working directory, accepts unbounded warnings, lets `// biome-ignore` comments accumulate without reasons, and runs whichever globally-installed version the developer last ran `npm i -g biome` with — diverging silently from CI. This spec pins the Biome version, the single committed config, the rule preset, the formatter ownership, the CI gate behavior, the suppression hygiene, and the migration path off ESLint+Prettier so `biome check` passing on a PR means the same thing on every machine and on the merge runner.

## Do

- Pin the Biome version exactly and run it through the project package manager.
- Configure Biome once in a committed `biome.json` (or `biome.jsonc`) with a `$schema` pinned to the same version.
- Run `biome check` (lint + format + organize imports) as the single CI gate; let warnings fail the build.
- Suppress an individual finding inline with `// biome-ignore <rule>: <reason>` citing the rule id and a real reason.
- Migrate off ESLint and Prettier in one PR using `biome migrate eslint` / `biome migrate prettier` and delete the old configs in the same change.

## Don't

- Run ESLint, Prettier, or a separate import-sort tool in parallel with Biome on overlapping files.
- Invoke a globally-installed `biome` from package scripts or hooks.
- Disable rules wholesale in `biome.json` to avoid fixing the underlying findings.
- Skip `biome check` in CI on the assumption that the local hook caught everything.
- Bump Biome silently in a routine dependency PR — breaking config changes need a dedicated PR with the migration output reviewed.

## References

- **spec** `prek` — sibling quality-layer spec for the hook runner that invokes Biome locally
- **spec** `trunk-code-quality` — sibling quality-layer spec; Biome is the JS/TS surface a Trunk-driven repo enables
- **external** `https://biomejs.dev/` — Biome home
- **external** `https://biomejs.dev/reference/configuration/` — `biome.json` configuration reference
- **external** `https://biomejs.dev/reference/cli/` — Biome CLI reference
- **external** `https://biomejs.dev/linter/rules/` — Biome lint rule catalogue
- **external** `https://biomejs.dev/guides/migrate-eslint-prettier/` — Migrating from ESLint and Prettier

## Rules

1. Adopt Biome as the single linter, formatter, and import organizer for every JavaScript, TypeScript, JSX, TSX, JSON, and JSONC file in the repository.
2. Pin the `@biomejs/biome` version as an exact `devDependency` in `package.json` (no `^`, `~`, or `>=`); do not depend on a floating range.
3. Configure Biome declaratively in a committed `biome.json` (or `biome.jsonc`) at the repository root; do not pass long CLI flag lists from `package.json` scripts.
4. Set the `$schema` field in `biome.json` to the schema URL matching the pinned Biome version (e.g. `https://biomejs.dev/schemas/<version>/schema.json`); do not leave `$schema` unset or pinned to a stale version.
5. Enable the `recommended` rule preset (`linter.rules.recommended: true`) and override deliberately with documented justifications; do not disable `recommended` to silence findings without justification.
6. Use Biome's formatter as the only JS/TS/JSX/TSX/JSON/JSONC formatter for the repo; do not run Prettier on the same file types in parallel.
7. Use Biome's linter as the only JS/TS/JSX/TSX linter for the repo; do not run ESLint on the same file types in parallel.
8. Enable `organizeImports.enabled = true` in `biome.json` and rely on Biome's import sort; do not run a separate import-sort plugin or `eslint-plugin-import` order rule.
9. Set `formatter.indentStyle`, `formatter.indentWidth`, `formatter.lineWidth`, `javascript.formatter.quoteStyle`, and `javascript.formatter.semicolons` explicitly in `biome.json`; do not rely on Biome's evolving defaults.
10. Run `biome check` (lint + format + organize imports) as a single CI gate on every pull request; treat any finding — error or warning — as a build failure.
11. Run Biome in CI in check-only mode (`biome check` without `--write`); do not let CI auto-apply fixes.
12. Run Biome through the project package manager (`pnpm exec biome`, `npm exec biome`, `bunx biome`, `yarn biome`); do not invoke a globally-installed Biome from package scripts, hooks, or CI.
13. Use `biome check --write` (and `--unsafe` when explicitly authorized for a fix class) only in local invocations and pre-commit hooks; do not run `--write` in CI.
14. Suppress an individual finding inline at the violation site with `// biome-ignore <rule>: <reason>` (or `/* biome-ignore-all <rule>: <reason> */` for a whole file) citing the rule id and a real reason; do not silence findings by removing the rule from `biome.json`.
15. Define path-scoped exceptions through Biome's top-level `overrides` array (tests, generated code, vendored sources) rather than disabling rules globally; do not push path-specific configuration into per-file ignore comments.
16. Configure `vcs.enabled = true` and `vcs.useIgnoreFile = true` so Biome respects the project's `.gitignore`; do not maintain a parallel ignore list under `files.ignore` that duplicates `.gitignore`.
17. Lint and format every file type Biome supports that exists in the repo (including JSON, JSONC, CSS where applicable); do not selectively skip a supported language without an `overrides` entry documenting why.
18. Wire Biome's LSP into editors via per-project configuration (the project's `@biomejs/biome` binary, not a global one) so contributors get the same diagnostics CI runs; do not rely on a global editor extension that bypasses the project's pinned version.
19. Bump Biome via a dedicated pull request that runs `biome migrate` for any breaking config changes, reviews the resulting `biome.json` diff, and updates the `$schema` URL in the same change; do not bundle a Biome upgrade into an unrelated dependency PR.
20. When migrating off ESLint and Prettier, run `biome migrate eslint --write` and `biome migrate prettier --write`, review the generated `biome.json`, and delete `.eslintrc*`, `.prettierrc*`, and the related `devDependencies` in the same PR; do not run both toolchains in parallel beyond the migration PR.
21. Invoke Biome from the project's hook runner (`prek`, `lefthook`, `husky`, etc.) on the changed-files set for fast pre-commit feedback; do not require contributors to install hooks by hand. (refs: prek)
