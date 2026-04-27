---
id: tailwind
layer: presentation
extends: []
---

# Tailwind CSS

## Purpose

Tailwind CSS is a utility-first CSS framework that generates styles from class names scanned in source files. Without shared conventions, teams abuse `@apply` to recreate semantic CSS, construct class names dynamically so the scanner misses them, and override the default scale instead of extending it — all of which undermine the utility-first model and produce bloated or broken builds. This spec establishes the rules that keep Tailwind maintainable, build outputs minimal, and the design system coherent.

## Do

- Write utility classes directly in markup as the primary styling method.
- Extend the design system through `@theme` (v4) or `theme.extend` (v3); never replace the default scale.
- Use arbitrary values `[value]` for one-off exceptions that don't belong in the theme.
- Use `@layer base` for default element styles and `@layer components` for reusable component classes.
- Safelist dynamically referenced classes with `@source inline()` (v4) or `safelist` (v3).

## Don't

- Construct class names by string concatenation — the scanner requires full, static class strings.
- Use `@apply` to recreate semantic CSS abstractions; reserve it for integrating with third-party HTML you cannot control.
- Override default theme scales entirely — extend them instead.
- Use the `theme()` CSS function in v4 — reference design tokens as CSS variables with `var(--token)`.
- Use deprecated v3 opacity utilities (`bg-opacity-*`, `text-opacity-*`) in v4 projects — use the slash modifier syntax instead.

## References

- **external** `https://tailwindcss.com/docs/adding-custom-styles` — @layer, @theme, arbitrary values, and @utility reference
- **external** `https://tailwindcss.com/docs/upgrade-guide` — v3 → v4 migration guide and breaking changes

## Rules

1. Write utility classes as complete, static strings in markup — never construct class names by concatenating string fragments, as the scanner cannot detect them. (refs: https://tailwindcss.com/docs/adding-custom-styles)
2. Add design tokens using `@theme` in v4 or `theme.extend` in v3; do not replace the default scale by overwriting top-level theme keys.
3. Place default HTML element styles in `@layer base`, reusable component abstractions in `@layer components`, and custom functional utilities in `@utility` (v4) or `@layer utilities` (v3).
4. Use arbitrary value syntax (`top-[117px]`, `text-[#1da1f2]`) for one-off values that are genuinely exceptions to the design system; do not add theme tokens for single-use values.
5. Do not use `@apply` to build component abstractions; use it only when styling HTML that cannot carry utility classes directly, such as third-party markup or generated content.
6. In v4 projects, reference design tokens in custom CSS as CSS variables (`var(--color-red-500)`) rather than the `theme()` function.
7. In v4 projects, replace `@tailwind base`, `@tailwind components`, and `@tailwind utilities` directives with a single `@import "tailwindcss"` statement.
8. Add classes that are assembled or loaded dynamically to the `@source inline()` directive (v4) or the `safelist` array (v3) so the scanner includes them in the build output.
9. Prefer `flex gap-*` or `grid gap-*` over `space-x-*`/`space-y-*` for long lists to avoid the selector-performance degradation of the sibling combinator.
10. In v4 projects, use the slash modifier syntax for opacity (`bg-black/50`, `text-white/75`) and remove any remaining `bg-opacity-*`, `text-opacity-*`, or `border-opacity-*` classes.
