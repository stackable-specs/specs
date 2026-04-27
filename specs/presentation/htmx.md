---
id: htmx
layer: presentation
extends: []
---

# htmx

## Purpose

htmx enables server-driven UI updates by extending HTML attributes to issue AJAX requests and swap response fragments into the DOM. Without shared conventions, teams return JSON instead of HTML, forget that pushed URLs must be independently navigable, and leave untrusted content exposed to attribute injection. This spec establishes the contracts that keep htmx-driven UIs correct, secure, and progressively enhanced.

## Do

- Respond to htmx requests with HTML fragments, not JSON.
- Ensure every URL pushed to history with `hx-push-url` also serves a full page on direct navigation.
- Use `Vary: HX-Request` on endpoints that render different content for htmx vs. full-page requests.
- Apply `hx-disable` around any element that renders untrusted or third-party HTML.
- Add `hx-history="false"` to pages or fragments containing sensitive data.

## Don't

- Return 3xx redirects when you need HX-* response headers — they are stripped on redirect.
- Inject htmx-attributed content via external JavaScript without calling `htmx.process(element)`.
- Push a URL to history that the server cannot serve as a complete page on direct navigation.
- Trust the client-side trigger as the sole authorization gate — always re-validate on the server.

## References

- **external** `https://htmx.org/docs/` — Full htmx attribute and configuration reference
- **external** `https://htmx.org/docs/#security` — Security section: hx-disable, history, CSP guidance

## Rules

1. Return HTML fragments (not JSON) from any endpoint that handles requests carrying the `HX-Request` header.
2. Any URL pushed to browser history via `hx-push-url` or `HX-Push-Url` must return a complete, standalone page when navigated to directly.
3. Set `Vary: HX-Request` on responses that serve different markup for htmx requests versus full-page requests to prevent cache collisions.
4. Escape all user-supplied content in server-rendered fragments before they are swapped into the DOM; do not rely on htmx to sanitize input.
5. Apply `hx-disable` to any element or subtree that renders untrusted or third-party HTML content to prevent attribute injection.
6. Add `hx-history="false"` to any page or fragment that contains sensitive data to exclude it from localStorage history snapshots.
7. Use `hx-boost` on standard `<a>` and `<form>` elements to progressively enhance navigation; ensure the target URLs remain functional without JavaScript.
8. Return `2xx` status codes when using HX-* response headers (`HX-Trigger`, `HX-Reswap`, `HX-Retarget`, etc.); 3xx redirects discard these headers before they reach the client.
9. Call `htmx.process(element)` after injecting content that contains htmx attributes via external JavaScript.
10. Use `hx-sync` to coordinate concurrent requests from the same section; do not allow competing requests to race and produce inconsistent DOM state.
