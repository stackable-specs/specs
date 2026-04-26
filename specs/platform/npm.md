---
id: npm
layer: platform
extends: []
---

# npm (Package Registry and Manifest)

## Purpose

npm is the shipment mechanism for the JavaScript and TypeScript ecosystem — both how third-party code arrives in a project and how internal packages reach their consumers. Inconsistent use produces real operational harm: a floating dependency range lets a malicious version ship to production, a missing or conflicting lockfile turns "works on my machine" into a daily problem, an absent `exports` field breaks every TypeScript consumer, a leaked `.env` inside a published tarball exposes production secrets, and publishing from a developer laptop forfeits provenance attestations that would otherwise prove who built what. This spec pins how packages are consumed and published so `npm install` produces identical trees everywhere, publishes are verifiable, and the package manifest accurately describes what was built.

## References

- **external** `https://www.npmjs.com/` — npm registry home
- **external** `https://docs.npmjs.com/` — npm documentation
- **external** `https://docs.npmjs.com/cli/v10/configuring-npm/package-json` — `package.json` reference
- **external** `https://nodejs.org/api/packages.html` — Node.js package entry points (`exports`, conditional exports)
- **external** `https://semver.org/` — Semantic Versioning 2.0.0
- **external** `https://docs.npmjs.com/generating-provenance-statements` — npm provenance statements

## Rules

1. Declare project metadata in `package.json` with explicit `name`, `version`, `description`, `license`, `repository`, and `type` fields populated.
2. Use scoped package names (`@org/name`) for organization-owned packages; do not claim generic unscoped names in the public registry.
3. Set `"type": "module"` for ESM-authored packages; do not ship a mixed CJS/ESM entry point without a discriminated `exports` map.
4. Follow Semantic Versioning (`MAJOR.MINOR.PATCH`) for every published version.
5. Increment the major version for any breaking change; do not ship breaking changes in minor or patch releases.
6. Publish prereleases under a non-`latest` dist-tag (`npm publish --tag beta|next|canary`); do not let prerelease versions become the default `latest`.
7. Record dependencies and devDependencies in `package.json` with explicit version ranges (`^X.Y.Z`, `~X.Y.Z`, or exact `X.Y.Z`); do not use `*`, `latest`, or git URLs in the committed manifest.
8. Commit exactly one lockfile per project (`package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`, or `bun.lock`); do not commit lockfiles from multiple package managers side by side.
9. Install dependencies in CI with a frozen-lockfile command (`npm ci`, `pnpm install --frozen-lockfile`, `yarn install --immutable`, or `bun install --frozen-lockfile`); do not run unfrozen installs on CI agents.
10. Run `npm audit --audit-level=high` (or the equivalent for the chosen package manager) in CI and treat findings at that level or above as build failures.
11. Define entry points in `package.json` `exports` with explicit `import`, `require`, and `types` conditions; do not rely solely on the legacy `main` or `module` fields for packages that ship multiple formats or TypeScript types.
12. Declare the type definitions entry via `exports[".types"]` (and a top-level `types` field for older tooling); do not publish a package without types if its sources are TypeScript.
13. Restrict the publish payload with the `files` field (or `.npmignore`) so only built artifacts ship; do not publish tests, source maps, CI configs, or unbuilt source unless they are intentionally part of the package.
14. Keep every repeatable command under `"scripts"` in `package.json`; do not document raw command invocations in the README that duplicate a scripted task.
15. Do not add `preinstall`, `install`, or `postinstall` scripts to published packages except when the package fundamentally requires a native build; document any such script's purpose in the README.
16. Publish from CI with OIDC-based authentication and `npm publish --provenance`; do not publish from a developer laptop.
17. Enable 2FA on every npm registry account permitted to publish a package.
18. Run `npm publish --dry-run` before every release and verify the resulting file list does not contain secrets, `.env` files, internal URLs, or unintended source.
19. Structure multi-package repos with the package manager's native workspaces feature (root `package.json` `workspaces` field, or `pnpm-workspace.yaml`); do not symlink packages manually or use file-path dependencies to reference packages within the same workspace.
