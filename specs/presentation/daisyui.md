---
id: daisyui
layer: presentation
extends:
  - tailwind
---

# daisyUI

## Purpose

daisyUI is a Tailwind CSS plugin that provides semantic component classes (btn, card, modal, etc.) and a CSS-variable-driven theming system. Without shared conventions, teams enable all 35 themes in production builds, hardcode colors instead of using theme tokens, and reach for raw CSS overrides before exhausting the built-in modifier system — producing bloated builds, inconsistent theming, and components that break when the design system changes. This spec establishes the rules that keep daisyUI components composable, themes controllable, and builds lean.

## Do

- Register daisyUI with `@plugin "daisyui"` and list only the themes your project uses.
- Apply themes via `data-theme` on `<html>` for global scope and on any child element for section-scoped overrides.
- Exhaust daisyUI modifier classes (`btn-primary`, `btn-outline`, `input-error`, etc.) before adding Tailwind utility overrides.
- Define custom themes using `@plugin "daisyui/theme"` and CSS variable assignments; do not patch component internals.
- Wire Tailwind's `dark:` prefix to your dark theme using `@custom-variant dark`.

## Don't

- Enable `themes: all` in production — include only the themes the project actually uses.
- Hardcode color values in component markup; use semantic theme tokens (`--color-primary`, `--color-base-100`, etc.).
- Override component styles by redefining daisyUI's internal classes — customize through theme variables or `@utility` extensions instead.

## References

- **spec** `tailwind` — Tailwind CSS utility-first rules this spec extends
- **external** `https://daisyui.com/docs/themes/` — Theming system, data-theme, and CSS variable reference
- **external** `https://daisyui.com/docs/customize/` — Component customization via modifiers, utilities, and CSS variables

## Rules

1. Add daisyUI to the CSS entry point as `@plugin "daisyui"` after `@import "tailwindcss"`. (refs: https://daisyui.com/docs/install/)
2. Declare only the themes your project uses in the plugin configuration (`themes: light --default, dark --prefersdark, cupcake`); do not use `themes: all` in production.
3. Designate exactly one theme as the default with the `--default` flag and one dark-mode theme with the `--prefersdark` flag in the plugin configuration.
4. Apply the active theme by setting `data-theme` on the `<html>` element; apply a scoped theme by setting `data-theme` on any child element.
5. Use daisyUI modifier classes (`btn-primary`, `btn-outline`, `input-error`, `card-bordered`, etc.) to vary component appearance before adding raw Tailwind utility overrides.
6. Define custom themes using `@plugin "daisyui/theme" {}` with CSS variable assignments (`--color-primary`, `--color-base-100`, `--radius-box`, etc.); do not patch component class internals.
7. Customize an existing built-in theme by redefining it with the same name and overriding only the variables you need; unspecified variables inherit their defaults.
8. Integrate Tailwind's `dark:` prefix with daisyUI themes by declaring `@custom-variant dark (&:where([data-theme=DARK_THEME_NAME], [data-theme=DARK_THEME_NAME] *))`.
9. Reference design tokens in custom CSS using CSS variables (`var(--color-primary)`) rather than hardcoded color values.
