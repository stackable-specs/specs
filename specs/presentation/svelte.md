---
id: svelte
layer: presentation
extends: []
---

# Svelte

## Purpose

Svelte 5's runes (`$state`, `$derived`, `$effect`, `$props`, `$bindable`) and SvelteKit's filesystem routing with server `load` and form `actions` are the framework's modern center of gravity — together they produce a small, statically analyzable, type-safe component model with first-class server-rendered data flow. The framework still accepts the legacy reactivity (`$:`, `export let`, `<slot>`, ad-hoc stores) for backwards compatibility, and mixing legacy and runes-style code in the same project yields files that look like two different frameworks side by side and forfeit the type narrowing that `$props<T>()` and `$bindable<T>()` provide. The same goes for `$derived` that performs side effects, missing keys on `{#each}`, fetching mutations from components instead of through SvelteKit form actions, and ignoring the compiler's accessibility warnings: each is "valid" Svelte but produces re-render surprises, lost list state, broken progressive enhancement, or quietly inaccessible UI. This spec pins the Svelte 5 idioms, the SvelteKit data-loading and mutation model, the build / lint / format / test toolchain, and the accessibility baseline so any `.svelte` file in the repo reads with consistent semantics and CI behaves as a real gate.

## References

- **spec** `typescript` — Svelte components in this repo follow the general TypeScript rules
- **spec** `vue` — sibling presentation-framework spec
- **spec** `react` — sibling presentation-framework spec
- **external** `https://svelte.dev/` — Svelte project home
- **external** `https://svelte.dev/docs/svelte/overview` — Svelte 5 documentation
- **external** `https://svelte.dev/docs/kit/introduction` — SvelteKit documentation
- **external** `https://github.com/sveltejs/eslint-plugin-svelte` — `eslint-plugin-svelte`
- **external** `https://github.com/sveltejs/prettier-plugin-svelte` — `prettier-plugin-svelte`
- **external** `https://testing-library.com/docs/svelte-testing-library/intro/` — Svelte Testing Library

## Rules

1. Author components as `.svelte` Single-File Components using Svelte 5 runes; do not write new code that relies on the legacy `$:` reactivity, `export let` props, or `<slot>` element.
2. Use TypeScript with `<script lang="ts">` for every component in TypeScript projects; do not write new components without `lang="ts"` when the project targets TypeScript.
3. Declare component props with `let { ... }: Props = $props()` and an explicit `Props` type; do not use `export let` for props in new code.
4. Declare two-way-bindable props with `$bindable<T>()`; do not synthesize two-way binding through manual prop + event pairs in new code.
5. Use `$state()` for reactive local state and `$state.raw()` when the value is large or should not be deeply tracked.
6. Use `$derived()` for derived state; do not store derived values in a `$state` updated from an `$effect`.
7. Use `$effect()` for side effects of reactive change and return a cleanup function for any subscription, timer, or listener; do not perform side effects (network requests, DOM writes, store mutations) inside a `$derived`.
8. Provide a stable, unique key on every `{#each}` block (`{#each items as item (item.id)}`); do not iterate without a key when the collection can be reordered, inserted into, or removed from.
9. Use `{#snippet}` and `{@render}` for content composition between components; do not use the legacy `<slot>` element in new code.
10. Encapsulate reusable DOM behavior in `use:action` directives; do not implement element-lifecycle behavior with `bind:this` plus `onMount` when an action expresses the same behavior locally.
11. Name component files in `PascalCase` (`UserCard.svelte`) and import them by the same name; do not mix `kebab-case` and `PascalCase` references for the same component.
12. Scope component styles with the default `<style>` block (Svelte scopes them automatically); use `:global()` only for documented design-system primitives, and never for component-local rules.
13. Build full applications with SvelteKit; do not assemble bespoke routing, SSR, or asset-pipeline tooling around plain Svelte for a new app.
14. Place routes under `src/routes/` using SvelteKit's filesystem convention; do not introduce a parallel client-side router for the app's primary navigation.
15. Load page data through SvelteKit `load` functions in `+page.ts`, `+page.server.ts`, `+layout.ts`, or `+layout.server.ts`; do not fetch primary page data inside component `$effect` blocks.
16. Type loaded data via the generated `PageData`, `LayoutData`, `PageLoad`, and `PageServerLoad` types from `./$types`; do not redeclare these shapes by hand.
17. Implement mutations as SvelteKit form `actions` invoked through `<form method="POST">` (enhanced with `use:enhance`); do not perform mutations by `fetch`-ing from a component without a corresponding action endpoint.
18. Keep secrets and server-only code in `+page.server.ts`, `+server.ts`, or modules under `$lib/server/`; do not import server-only code from client-reachable modules.
19. Treat Svelte compiler accessibility warnings (`a11y-*`) as errors in CI by setting `compilerOptions.warningFilter` to escalate them, or by configuring the lint stage to fail on them; do not silence them with file-level disables.
20. Use semantic HTML elements (`<button>`, `<nav>`, `<main>`, `<label>`, `<dialog>`, etc.) for their intended roles; add `aria-*` attributes only to supplement behavior HTML cannot express.
21. Provide an `alt` attribute on every `<img>`; use `alt=""` for purely decorative images so assistive tech can skip them.
22. Lint with `eslint-plugin-svelte` and `svelte-check` in CI; treat findings and type errors reported by `svelte-check` as build failures.
23. Format `.svelte` files with Prettier plus `prettier-plugin-svelte` using a single committed config; CI must fail on unformatted files.
24. Test components with Vitest plus `@testing-library/svelte`, and write end-to-end tests with Playwright; do not add Jest or Cypress as a parallel framework in new modules.
