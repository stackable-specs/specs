---
id: nextjs
layer: presentation
extends:
  - typescript
---

# Next.js

## Purpose

Next.js is a full-stack React framework whose surface is wide enough to do almost anything and sharp enough to do most of those things wrong by accident. Marking a layout `"use client"` so one button can have an `onClick` ships the entire subtree to the browser as JavaScript. Importing a database client from a file that gets pulled into a Client Component bundles the connection string into the public asset. A Server Action without runtime validation accepts whatever `FormData` a hostile client posts, and a Server Action without an authorization re-check trusts whatever the UI happened to render. Forgetting `revalidatePath` after a mutation serves stale data; setting `dynamic = "force-dynamic"` on a page that didn't need it kills caching for everyone; setting `force-static` on one that did serves yesterday's content. Heavy work in `middleware.ts` runs on every request including assets. This spec pins the App Router as the substrate, the Server-by-default rendering model, the server/client module separation, the per-route caching declarations, the Server Action / Route Handler validation and authorization rules, and the framework-native primitives for images, fonts, and links — so a Next.js app remains fast, secure, and predictable as it grows past the create-next-app demo.

## References

- **spec** `typescript` — language-layer TypeScript spec this refines
- **external** `https://nextjs.org/` — Next.js home
- **external** `https://nextjs.org/docs/app` — App Router documentation
- **external** `https://nextjs.org/docs/app/getting-started/server-and-client-components` — Server vs Client Components
- **external** `https://nextjs.org/docs/app/getting-started/server-actions-and-mutations` — Server Actions and mutations
- **external** `https://nextjs.org/docs/app/api-reference/file-conventions/route-segment-config` — Route segment config (`dynamic`, `revalidate`, `fetchCache`)
- **external** `https://nextjs.org/docs/app/building-your-application/caching` — Caching model
- **external** `https://nextjs.org/docs/app/api-reference/file-conventions/middleware` — Middleware reference
- **external** `https://nextjs.org/docs/app/api-reference/components/image` — `next/image`
- **external** `https://nextjs.org/docs/app/api-reference/components/font` — `next/font`
- **external** `https://nextjs.org/docs/app/api-reference/components/link` — `next/link`
- **external** `https://www.npmjs.com/package/server-only` — `server-only` import guard
- **external** `https://www.npmjs.com/package/client-only` — `client-only` import guard

## Rules

1. Use the App Router (`app/` directory) for new projects; do not start a new project on the legacy Pages Router.
2. Pin the Next.js version as an exact `dependency` in `package.json` (no `^` or `~`); upgrade deliberately via a dedicated PR with the release notes reviewed.
3. Default every component to a Server Component and place `"use client"` only at the smallest file that requires interactivity (state, effects, browser APIs, event handlers); do not mark a layout or large page component client-only when only one leaf needs it.
4. Do not import server-only modules (`fs`, `child_process`, database clients, secret-bearing config) from a Client Component or from any module reachable through a `"use client"` boundary.
5. Tag server-only modules with the `server-only` package and client-only modules with `client-only`; do not rely on bundler convention alone to keep them separated.
6. Read secrets and server-side configuration from non-`NEXT_PUBLIC_*` environment variables in Server Components, Route Handlers, or Server Actions only; do not expose secrets via `NEXT_PUBLIC_*` variables.
7. Validate every Server Action and Route Handler input with a runtime schema validator (e.g. Zod, Valibot); do not trust `formData` or request-body shapes inferred from TypeScript alone.
8. Authorize every Server Action and Route Handler by re-checking session and permissions inside the handler; do not rely on UI conditional rendering or middleware as the sole authorization gate.
9. Mutate data through Server Actions or Route Handlers and call `revalidatePath` / `revalidateTag` for every cache surface the mutation invalidates; do not return stale cached data after a write.
10. Declare per-route caching with explicit `revalidate`, `dynamic`, and `fetchCache` segment options (or per-`fetch` cache options); do not rely on framework-default caching semantics that have changed between major versions.
11. Set `dynamic = "force-static"` (or supply `generateStaticParams`) for pre-renderable routes and `dynamic = "force-dynamic"` only when the route genuinely depends on per-request data.
12. Provide `loading.tsx` and `error.tsx` for every route segment that performs server data fetching; do not let a suspense-bounded segment render a blank fallback or rethrow uncaught errors past the segment.
13. Use `next/image` for raster images served by the app, with explicit `width`, `height`, and `alt` props; do not use raw `<img>` for images the app ships.
14. Use `next/font` (or the equivalent self-hosted-font integration) for web fonts; do not include a `<link rel="stylesheet">` to a third-party font CDN in production.
15. Use `next/link` for in-app navigation; do not use raw `<a href="/internal/…">` for routes the app owns.
16. Keep `middleware.ts` limited to auth checks, redirects, and rewrites, with an explicit `matcher` that scopes the routes it runs on; do not perform heavyweight database queries or external HTTP calls from middleware.
17. Do not set `typescript.ignoreBuildErrors` or `eslint.ignoreDuringBuilds` to `true` in `next.config.*`; let `next build` enforce both gates. (refs: typescript)
18. Lint with `eslint-config-next` (or the official Next.js ESLint plugin) on every file in `app/` and `components/`; do not disable Next.js-specific rules without an inline comment explaining why.
