---
id: mkdocs
layer: presentation
extends: []
---

# MkDocs

## Purpose

MkDocs converts a directory of Markdown into a navigable documentation site cheaply and reliably — but only when the configuration, navigation, theme version, and plugin set are pinned, when `strict` mode is on, when the CI build fails the same way the local build fails, and when renaming a page does not silently delete a URL someone bookmarked. Left to defaults, MkDocs accepts broken `[link](other.md)` references with a warning that nobody reads, lets the Material theme version float so the navigation reorganizes itself between deploys, falls back to alphabetical filename ordering when `nav:` is omitted, and publishes a `latest/` site that has drifted three versions past the released software because no team ever set up `mike`. Without `mkdocstrings` (or its language equivalent) the API reference is hand-pasted from generator output and goes stale on the next refactor; without `mkdocs-redirects` every rename is an inbound 404; without a CI build gate, broken links land on `main` and ship to the public site. This spec pins the configuration shape, the version-pinning of MkDocs / theme / plugins, the navigation discipline, the strict-build CI gate, link-checking, asset hygiene, generated API reference, redirects on rename, and versioned publishing — so a docs site stays trustworthy for the consumers who already linked to it.

## References

- **spec** `typedoc` — sibling presentation-layer spec for TypeScript API reference (which `mkdocstrings` is the Python analog of)
- **spec** `uv` — platform-layer spec for pinning `mkdocs` and plugins as Python dependencies
- **external** `https://www.mkdocs.org/` — MkDocs documentation
- **external** `https://github.com/mkdocs/mkdocs` — MkDocs source repository
- **external** `https://squidfunk.github.io/mkdocs-material/` — Material for MkDocs theme
- **external** `https://mkdocstrings.github.io/` — `mkdocstrings` (auto API reference)
- **external** `https://github.com/mkdocs/mkdocs-redirects` — `mkdocs-redirects`
- **external** `https://github.com/jimporter/mike` — `mike` (versioned docs publishing)
- **external** `https://facelessuser.github.io/pymdown-extensions/` — PyMdownx Markdown extensions
- **external** `https://lychee.cli.rs/` — `lychee` link checker

## Rules

1. Author docs as Markdown under a single `docs/` directory at the project root; do not scatter documentation across the repo.
2. Configure MkDocs via a single `mkdocs.yml` at the project root; do not split configuration across multiple files except via documented `INHERIT:` chains.
3. Pin MkDocs and every theme/plugin to an exact version in the project's dependency manifest (`pyproject.toml` under uv / Poetry, or a pinned `requirements.txt`); do not install `mkdocs` and `mkdocs-material` without version constraints. (refs: uv)
4. Set `strict: true` in `mkdocs.yml` and run `mkdocs build` in CI on every PR so broken links, missing nav entries, and missing files fail the build; do not silence the failure with `--no-strict`.
5. Run a link checker (`lychee`, `linkchecker`, or `mkdocs-linkcheck`) against the built site in CI; do not merge changes that introduce new broken external links.
6. Declare the navigation explicitly with `nav:` in `mkdocs.yml`; do not rely on alphabetical filename ordering as the published navigation.
7. Place each page's H1 (`# Title`) at the top of the Markdown file; do not duplicate the title between filename and a frontmatter `title:` field without a single project-wide convention.
8. Pin the documentation theme (Material for MkDocs, ReadTheDocs, or equivalent) and its version in the project's dependency manifest; do not let the theme version float between builds.
9. Enable theme features (e.g. Material `navigation.tabs`, `search.suggest`, `content.code.copy`) explicitly in `mkdocs.yml`'s `theme.features:` list; do not leave theme features at upstream defaults that change between versions.
10. Set `site_url`, `repo_url`, `edit_uri`, and `site_description` in `mkdocs.yml`; do not publish a docs site without canonical URL metadata (sitemap, OG tags, and "Edit this page" links depend on it).
11. Enable Markdown extensions explicitly in `markdown_extensions:` (admonitions, fenced code, tables, attribute lists, definition lists, footnotes, PyMdownx as needed); do not rely on a plugin's side effects to register extensions.
12. Tag every fenced code block with a language identifier (` ```python `, ` ```bash `, ` ```yaml `); do not commit unfenced code blocks or untagged ` ``` ` blocks.
13. Reference assets (images, attachments) via paths relative to the page's `.md` file under `docs/`; do not link to assets outside `docs/` or hotlink images from URLs the team does not control.
14. Generate API reference sections from source code via `mkdocstrings` (Python) or the language-appropriate plugin; do not paste auto-generated API docs into hand-edited Markdown. (refs: typedoc)
15. Maintain redirects with `mkdocs-redirects` whenever a page is renamed or removed; do not break inbound links by deleting a published URL without a redirect entry.
16. Version published docs with `mike` (or an equivalent versioned-publishing plugin) so each released version is reachable at a stable URL path and the default alias points at the latest released version; do not maintain only a single "latest" site that drifts past the released software version.
17. Build and deploy docs from CI on every push to the default branch (and on every release tag for versioned sites); do not publish docs from a developer laptop.
