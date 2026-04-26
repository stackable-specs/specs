---
id: pinia
layer: presentation
extends: [vue]
---

# Pinia

## Purpose

Pinia is the official store library for Vue 3 and the canonical place to keep cross-component state — but its ergonomics make it equally easy to misuse. Stores defined in options syntax look like Vuex with a fresh coat of paint, destructuring a store without `storeToRefs` silently loses reactivity, mutating state from inside a component bypasses the action boundary that makes flows debuggable, a single "app" store accumulates unrelated slices until tests have to spin up the universe to assert on a checkbox, and missing TypeScript narrowing turns a typed framework back into runtime guesswork. This spec pins how Pinia stores are defined, scoped, typed, mutated, composed, persisted, and tested so cross-component state has a single recognizable shape across the codebase, components stay decoupled from store internals, and reactivity holds end-to-end.

## References

- **spec** `vue` — parent presentation-framework spec; Pinia is a Vue-specific refinement
- **spec** `typescript` — Pinia stores follow the general TypeScript rules
- **external** `https://pinia.vuejs.org/` — Pinia documentation home
- **external** `https://pinia.vuejs.org/core-concepts/` — Pinia core concepts
- **external** `https://pinia.vuejs.org/cookbook/composing-stores.html` — Composing stores
- **external** `https://pinia.vuejs.org/cookbook/testing.html` — Testing Pinia stores
- **external** `https://prazdevs.github.io/pinia-plugin-persistedstate/` — `pinia-plugin-persistedstate`

## Rules

1. Use Pinia (not Vuex) for shared cross-component state in every Vue 3 module; do not introduce or extend Vuex stores in new code.
2. Define every store with `defineStore('<id>', () => { ... })` using the setup-syntax function form; do not define new stores with the options-object form.
3. Name stores `useXxxStore` and place each in its own file named after the store id (`stores/user.ts` exporting `useUserStore`); do not co-locate multiple stores in a single file.
4. Type each store explicitly in TypeScript — annotate `ref<T>()`, `reactive<T>()`, and action parameter / return types; do not rely on inferred `any` for state slices.
5. Scope each store to a single domain (auth, cart, current document, feature flags); do not create an "app" or "global" store that accumulates unrelated slices.
6. Mutate state only inside the store's setup function — through `ref`/`reactive` writes inside actions, or through `store.$patch` from a calling action; do not assign to store state from a component or composable.
7. Use `store.$patch({...})` (or `$patch(state => { ... })`) for any update that touches more than one field together; do not perform multi-field updates as a sequence of individual writes.
8. Encapsulate async work (network requests, IO, multi-step flows) as actions on the store; do not orchestrate multi-step state changes from inside a component.
9. Compose stores by calling other stores' `useXxxStore()` inside actions or getters; do not pass store instances around as props or function arguments.
10. Use `storeToRefs(store)` when destructuring reactive state and getters in components; do not destructure state directly off the store object (the destructured fields lose reactivity).
11. Expose actions on the destructured store object directly (actions retain their binding); do not wrap actions in `storeToRefs`.
12. Use `computed()` inside the setup function for derived state; do not store derived values in a `ref` updated by a `watch`.
13. Provide an explicit `reset()` action on every store whose lifecycle includes logout, tenant switch, or route exit; do not rely on the options-style `$reset()` helper from a setup-syntax store (it does not exist there).
14. Persist only the minimum slice that must survive a reload (auth tokens, user preferences) using `pinia-plugin-persistedstate` or an equivalent plugin; do not persist server-cached collections or transient UI state.
15. Treat server-fetched data and view state as separate concerns — keep cache freshness in the data slice, keep ephemeral view state (selection, filter input) in the consuming component or a separate UI store; do not commingle the two in one ref.
16. Implement cross-cutting behavior (logging, persistence, error reporting) as a Pinia plugin registered once on `createPinia()`; do not duplicate the same wiring inside individual stores.
17. Do not reach into another store's internals from a component — call its actions and read its (`storeToRefs`-wrapped) state; do not import internal refs by name from another store's module.
18. Test stores with Vitest by calling `setActivePinia(createPinia())` in a `beforeEach` so each test starts with a fresh root; do not let store state leak across tests.
19. Mock store collaborators in component tests via `createTestingPinia()` from `@pinia/testing` with `stubActions: true` by default; do not hand-roll a fake module that diverges from the real store's surface.
