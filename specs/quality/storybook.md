---
id: storybook
layer: quality
extends: []
---

# Storybook

## Purpose

Storybook is a component-state catalog that doubles as a verification surface — interaction tests, visual regression, accessibility checks — and as living documentation published per PR. The value only materializes when every component has stories covering its real states, the stories are wired into test runners and visual-diff pipelines, and the Storybook build is reviewable by teammates without running it locally. The common failure mode is treating Storybook as a demo playground: scattered formats, stale fixtures, stories that call real APIs, and no enforced tests — which degrades into a second codebase that "documents" what the real code used to do six months ago. This spec pins the story format, coverage expectations, and test wiring so Storybook stays a verification tool rather than dusty marketing material.

## References

- **spec** `react` — React components whose states Storybook verifies
- **external** `https://storybook.js.org/` — Storybook home
- **external** `https://storybook.js.org/docs` — Storybook documentation
- **external** `https://storybook.js.org/docs/writing-stories` — Writing stories (Component Story Format 3)
- **external** `https://storybook.js.org/docs/writing-tests` — Writing tests with `play` functions
- **external** `https://github.com/storybookjs/storybook` — Storybook source repository
- **external** `https://github.com/mswjs/msw-storybook-addon` — MSW addon for mocking network in stories

## Rules

1. Colocate each `*.stories.tsx` file next to the component it describes (same directory as the component source).
2. Create at least one story for every component exported from a shared, design-system, or public package.
3. Write stories in Component Story Format 3 (CSF3): a `default` export for `meta` and named `StoryObj` exports for each story.
4. Drive story variation through `args` and `argTypes` controls; do not duplicate a story by hardcoding different prop values in separate story bodies.
5. Enable autodocs by setting `tags: ['autodocs']` on the `meta` export for every story file unless generated docs are explicitly unwanted.
6. Name stories after user-visible states (`Default`, `Loading`, `Empty`, `Error`, `Disabled`, `LongContent`); do not name stories after implementation details (`WithFlagXTrue`).
7. Register `@storybook/addon-a11y` in `.storybook/main.ts` and treat any WCAG AA violation surfaced by the addon as a failing check in CI.
8. Write interaction tests inside `play` functions using `userEvent` and `expect` imported from `@storybook/test`.
9. Mock network requests in stories with MSW (via `msw-storybook-addon`) or an equivalent in-browser mock layer; do not let stories call real backend endpoints.
10. Run story tests in CI via `@storybook/test-runner` or the Vitest Storybook addon and treat failures as build failures.
11. Run visual regression tests against published Storybook builds (Chromatic, Percy, Loki, or Playwright visual comparisons) and gate merges on passing diffs.
12. Publish a Storybook build per pull request to a reviewable preview URL; do not require reviewers to run Storybook locally to inspect changes.
13. Do not put business logic, data fetching, or routing flow inside stories — stories render display states, they do not encode workflows.
14. Configure Storybook in `.storybook/main.ts` and global behavior (decorators, parameters, themes) in `.storybook/preview.tsx`; do not modify Storybook behavior through runtime monkey-patching.
15. Provide globals used by components (theme, locale, router, `QueryClient`, etc.) through global decorators in `preview.tsx`; do not require each story to set these up individually.
16. Pin `storybook` and its framework package (`@storybook/react-vite`, `@storybook/nextjs`, `@storybook/vue3-vite`, etc.) to exact versions in `package.json`; upgrade the set together via `npx storybook upgrade`.
17. Use the Storybook framework that matches the application's build tooling (e.g. a React + Vite application uses `@storybook/react-vite`, not a mismatched framework package).
18. Import the real component into stories; do not create a parallel or simplified implementation inside the story file.
