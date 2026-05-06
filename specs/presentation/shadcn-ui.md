---
id: shadcn-ui
layer: presentation
extends:
  - tailwind
---

# shadcn/ui

## Purpose

shadcn/ui is a copy-into-your-project component system built on Tailwind CSS and Radix UI primitives. Unlike traditional component libraries, components live in your source tree (`components/ui/`) giving you full ownership and the freedom to modify them. Without shared conventions, teams bypass the CLI and hand-copy stale source, hardcode raw colors instead of semantic tokens, wrap components in extra abstraction layers rather than editing them directly, and skip the `cn()` utility — producing duplicated variants, theming inconsistencies, and brittle wrappers that drift from upstream. This spec establishes the rules that keep components current, the theme coherent, and customization first-class.

## Do

- Add components via `npx shadcn@latest add <component>` so they land in `components/ui/` with up-to-date source.
- Use `cn()` from `lib/utils.ts` to merge and conditionally apply Tailwind class names inside all component files.
- Define all theme tokens under `:root` and `.dark` in the global CSS file; reference them as semantic utilities (`bg-primary`, `text-foreground`, `border-destructive`).
- Customize component behavior by editing the file in `components/ui/` directly.
- Enable `cssVariables: true` in `components.json`.

## Don't

- Install shadcn/ui as a versioned npm dependency — it is a code-copy system, not a package.
- Wrap components in an extra React component layer solely to modify styles or props; edit the source instead.
- Hardcode raw color utilities (`bg-blue-500`, `text-gray-900`) for theme-controlled colors; use semantic tokens.
- Define theme tokens inside component files; they belong only in the global CSS file.

## References

- **spec** `tailwind` — Tailwind CSS utility-first rules this spec extends
- **external** `https://ui.shadcn.com/docs` — Getting started, CLI reference, and component catalogue
- **external** `https://ui.shadcn.com/docs/theming` — CSS variable theming system and dark mode strategy

## Rules

1. Initialize shadcn/ui with `npx shadcn@latest init` and add components with `npx shadcn@latest add <component>`; do not hand-copy component source or install a versioned shadcn package. (refs: https://ui.shadcn.com/docs)
2. Keep all generated component files in the `components/ui/` directory as specified by `components.json`; do not scatter them across unrelated directories.
3. Set `cssVariables: true` in `components.json` so all components resolve styles through CSS variables rather than inlined color values.
4. Define all theme tokens in the global CSS file (e.g., `app/globals.css`) under `:root` for light mode and `.dark` for dark mode; do not define theme tokens inside component files.
5. Map CSS variables to Tailwind utilities via `@theme inline` declarations (e.g., `--color-primary: var(--primary)`) so components can reference them as standard utility classes.
6. Use the `cn()` utility from `lib/utils.ts` to merge and conditionally apply Tailwind class names inside every component file.
7. Reference semantic token utilities (`bg-primary`, `text-foreground`, `border-destructive`, `text-muted-foreground`, etc.) for theme-controlled colors; do not use raw color utilities (`bg-blue-500`) in component markup.
8. Customize a component by editing its file in `components/ui/` directly; do not wrap it in an additional React component purely to alter its styles or props.
9. Compose new UI surfaces from existing shadcn/ui primitives rather than building parallel implementations from scratch.
10. When pulling upstream component updates, run `npx shadcn@latest add <component>` and review the diff to merge any local customizations before accepting the new source.
