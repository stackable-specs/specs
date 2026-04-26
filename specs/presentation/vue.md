---
id: vue
layer: presentation
extends: []
---

# Vue.js

## Purpose

Vue 3's Composition API with `<script setup>` is the framework's modern center of gravity — reactivity primitives (`ref`, `reactive`, `computed`, `watch`), typed `defineProps` / `defineEmits` / `defineModel`, composables, and Single-File Components together produce a small, testable, type-safe component model. The framework still ships the Options API, mixins, runtime-string templates, and array-syntax props for backwards compatibility — and mixing them with Composition-style code in the same project yields files that read like two different frameworks side by side and forfeit the type guarantees that `<script setup lang="ts">` provides. The same goes for `computed` that performs side effects, undeclared emits, missing or index-based `:key` on `v-for`, and `v-for` combined with `v-if` on a single element: each is "valid" Vue but produces mysterious re-renders, lost list state, or evaluation-order surprises that only appear under specific data shapes. This spec pins the Vue 3 idioms, the build / lint / format / test toolchain, and the accessibility baseline so any `.vue` file in the repo reads with consistent semantics and CI behaves as a real gate.

## References

- **spec** `typescript` — Vue components in this repo follow the general TypeScript rules
- **spec** `react` — sibling presentation-framework spec
- **external** `https://vuejs.org/` — Vue.js home
- **external** `https://vuejs.org/guide/introduction.html` — Vue 3 guide
- **external** `https://vuejs.org/style-guide/` — Vue style guide
- **external** `https://eslint.vuejs.org/` — `eslint-plugin-vue`
- **external** `https://pinia.vuejs.org/` — Pinia state management
- **external** `https://router.vuejs.org/` — Vue Router

## Rules

1. Author components as Single-File Components (`.vue` files) using the Composition API with `<script setup>`; do not use the Options API or non-setup `<script>` blocks in new code.
2. Type Vue components in TypeScript with `<script setup lang="ts">` in TypeScript projects; do not write Vue 3 components without `lang="ts"` when the project targets TypeScript.
3. Declare props with `defineProps<{...}>()` using an explicit TypeScript interface; do not use the runtime array-or-object props syntax in `<script setup>`.
4. Declare emitted events with `defineEmits<{...}>()` using explicit signatures; do not emit events that have not been declared.
5. Use `defineModel<T>()` for two-way bindings on Vue 3.4+; do not implement `v-model` by hand as a `modelValue` prop plus `update:modelValue` emit in new code.
6. Use `ref()` for reactive primitives and single values, and `reactive()` for object / collection containers; access ref values through `.value` or destructure with `toRefs` when passing to functions.
7. Use `computed()` for derived state; do not store derived values in a `ref` updated by a `watch`.
8. Use `watch()` or `watchEffect()` for side effects of reactive change; do not perform side effects (network requests, DOM writes, mutations) inside a `computed`.
9. Extract reusable stateful logic into composables — functions named `useXxx` in a `composables/` directory; do not use mixins in new code.
10. Provide a stable, unique `:key` on every `v-for`; do not use the loop index as a key for collections that can be reordered, inserted into, or removed from.
11. Do not combine `v-for` and `v-if` on the same element; filter the source collection in a `computed`, or wrap the iteration in a parent element.
12. Name components in `PascalCase` in source and reference them as `PascalCase` in templates throughout the project; do not mix `PascalCase` and `kebab-case` references for the same component.
13. Scope component-local styles with `<style scoped>` or CSS modules; do not introduce global styles from a component file without a documented reason.
14. Use Pinia for cross-component state; do not add Vuex to new modules.
15. Use Vue Router with typed route definitions and the typed `useRouter` / `<RouterLink>` APIs; do not navigate by hand-built URL strings constructed inside components.
16. Provide app-level globals (theme, i18n, router, Pinia stores) via plugin install or `app.provide`; do not require every component to import a singleton.
17. Wrap async components and async-data subtrees in `<Suspense>` with a fallback; do not replicate loading-state booleans at every data-consuming call site.
18. Register a global error handler via `app.config.errorHandler` and use `onErrorCaptured` at meaningful subtree roots; do not rely on browser-level uncaught-error handlers for component failures.
19. Use semantic HTML elements (`<button>`, `<nav>`, `<main>`, `<label>`, `<dialog>`, etc.) for their intended roles; add `aria-*` attributes only to supplement behavior HTML cannot express.
20. Provide an `alt` attribute on every `<img>`; use `alt=""` for purely decorative images so assistive tech can skip them.
21. Lint with `eslint-plugin-vue` (`flat/recommended` or higher) and `eslint-plugin-vuejs-accessibility` in CI; treat findings as build failures.
22. Format `.vue` files with Prettier using a single committed config; CI must fail on unformatted files.
23. Test components with Vitest plus Vue Test Utils (or Testing Library Vue); do not add Jest as a parallel framework in new modules.
