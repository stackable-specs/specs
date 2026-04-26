---
id: docusaurus
layer: presentation
extends:
  - typescript
---

# Docusaurus

## Purpose

Docusaurus's value comes from a small, opinionated contract: site configuration in a single typed file, navigation in a single typed sidebar file, content written as MDX with declared front matter, and a build that turns broken links and missing translations into hard failures. Held to that contract, a Docusaurus site is reproducible, versionable, and search-indexable, and a docs PR has the same review semantics as a code PR. Held loosely, the same project quietly degrades into a CMS: the dev server is left running as production, broken-links are downgraded from `throw` to `warn` until they're invisible, page ordering drifts between filename prefixes / `_category_.json` / `sidebars.ts`, `versioned_docs/` is hand-edited so the published 2.x site no longer matches any tag, translations land in ad-hoc folders Crowdin cannot read, Algolia DocSearch keys ship in `docusaurus.config.ts`, and `@docusaurus/*` package versions drift apart so plugin upgrades fail mysteriously. This spec pins the configuration shape, content authoring conventions, versioning and i18n workflow, broken-link gate, deployment posture, and dependency-pinning discipline so a Docusaurus repo stays a single source of truth for documentation rather than a Next.js project in disguise.

## Do

- Configure the site in `docusaurus.config.ts` and navigation in `sidebars.ts` — TypeScript, single source of truth.
- Pin `docusaurus` and every `@docusaurus/*` package to the same exact version; upgrade in one coordinated PR.
- Author content as `.mdx` when the page embeds React, `.md` when it does not, with explicit front matter.
- Set `onBrokenLinks: 'throw'` and `onBrokenMarkdownLinks: 'throw'` so broken references fail the build.
- Snapshot stable releases with `docusaurus docs:version <X>` before publishing breaking changes.
- Manage translations under `i18n/<locale>/` and refresh source strings with `docusaurus write-translations`.
- Load DocSearch keys and other third-party credentials from build-time environment variables.

## Don't

- Run `docusaurus start` (the dev server) as a production process — `docusaurus serve` the built output instead.
- Edit files under `versioned_docs/` or `versioned_sidebars/` by hand to "fix" a published version.
- Lower `onBrokenLinks` to `warn` or `ignore` to make a red build green.
- Mix preset-classic with parallel `@docusaurus/plugin-content-docs` instances unless deliberately running multi-instance docs.
- Commit Algolia DocSearch keys, analytics IDs, or other secrets in `docusaurus.config.ts`.
- Stand up a competing static-site generator (Next.js, Nextra, MkDocs, VitePress, Sphinx) alongside Docusaurus for the same documentation surface.

## References

- **spec** `typescript` — language baseline this spec extends
- **external** `https://docusaurus.io/` — Docusaurus home
- **external** `https://docusaurus.io/docs` — Docusaurus documentation
- **external** `https://docusaurus.io/docs/configuration` — `docusaurus.config.ts` reference
- **external** `https://docusaurus.io/docs/sidebar` — sidebar configuration
- **external** `https://docusaurus.io/docs/versioning` — docs versioning workflow
- **external** `https://docusaurus.io/docs/i18n/introduction` — internationalization setup
- **external** `https://mdxjs.com/` — MDX (Markdown + JSX) format
- **external** `https://docsearch.algolia.com/` — Algolia DocSearch integration

## Rules

1. Pin `docusaurus` and every `@docusaurus/*` package to the same exact version in `package.json` (no `^` / `~`); upgrade them together in a single PR.
2. Configure the site in `docusaurus.config.ts` (TypeScript); do not maintain `docusaurus.config.js` and `docusaurus.config.ts` in parallel. (refs: typescript)
3. Configure sidebar navigation in `sidebars.ts` committed to source control; do not generate sidebars at deploy time from external state.
4. Author a page as `.mdx` when it embeds React components or JSX, and as `.md` when it does not; do not name a plain-Markdown file `.mdx` for uniformity.
5. Place documentation pages under `docs/` (or the configured `path`) and route them at the project's declared `routeBasePath`; do not scatter docs across multiple top-level directories without a documented convention.
6. Declare each page's `id`, `title`, `sidebar_position`, `slug`, and `description` in front matter when ordering, slug, or sidebar position must be stable; do not depend on filename ordering for published navigation.
7. Use `_category_.json` only for folder-level metadata that `sidebars.ts` cannot express; when both define the same item, the `sidebars.ts` declaration is authoritative.
8. Set `onBrokenLinks: 'throw'` and `onBrokenMarkdownLinks: 'throw'` in `docusaurus.config.ts`; do not lower either to `warn` or `ignore` to make a failing build pass.
9. Snapshot a stable version with `docusaurus docs:version <X>` before publishing a release; do not edit files under `versioned_docs/` or `versioned_sidebars/` by hand after they are written.
10. Manage translations under `i18n/<locale>/` and refresh source strings with `docusaurus write-translations`; do not hand-create translation files outside that layout.
11. Run `docusaurus build` in CI on every PR and treat any failure (including broken-link or i18n failures) as a merge blocker.
12. Serve the production site with `docusaurus serve` over the output of `docusaurus build`; do not run `docusaurus start` (the dev server) as a production process.
13. Load Algolia DocSearch credentials, analytics IDs, and other third-party API keys from environment variables read at build time; do not commit them to `docusaurus.config.ts` or any tracked file.
14. Declare every plugin, preset, and theme in `presets`, `plugins`, or `themes` of `docusaurus.config.ts` with explicit options; do not rely on undocumented zero-config defaults for behavior the team depends on.
15. Place swizzled or shadowed theme components under `src/theme/` and re-verify them on every `@docusaurus/*` upgrade; do not edit framework files in `node_modules` to customize the theme.
16. Do not run a competing static-site generator (Next.js, Nextra, MkDocs, VitePress, Sphinx) against the same documentation tree in the same repository.
