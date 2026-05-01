---
name: render-spec
description: "Render a JSON spec object into the canonical stackable-specs markdown format. Use when the user supplies JSON (or a JSON file) matching the spec schema — fields id, layer, purpose, rules, optional extends, optional references — and asks to convert, render, format, emit, or output it as a .md spec file. Also use when given a raw JSON blob and asked to 'turn this into a spec' or 'render this as markdown'. Produces a file with YAML frontmatter (id, layer, extends) and markdown sections (title, Purpose, References, Rules) suitable to drop into specs/LAYER/ID.md."
---

# Render Spec

Convert a schema-conforming JSON spec object into a markdown spec file that matches the stackable-specs output shape. This is the inverse of reading a spec: input is structured JSON, output is the canonical `.md`.

## Input shape

A JSON object with these fields (see `assets/schema.json` for the canonical schema):

- `id` (string, required) — kebab-case identifier
- `layer` (string, required) — one of: `language`, `platform`, `interface`, `presentation`, `delivery`, `observability`, `security`, `practices`, `quality`
- `purpose` (string, required) — one paragraph of prose
- `rules` (array, required) — each item `{ statement: string, references?: string[] }`
- `extends` (array of strings, optional) — parent spec ids
- `references` (array, optional) — each item `{ type: "spec"|"adr"|"bdr"|"external", ref: string, description?: string }`

## Output shape

A single markdown document in this exact order and form:

```
---
id: <id>
layer: <layer>
extends: [<comma-separated ids> | omitted if empty]
---

# <Title derived from id>

## Purpose

<purpose paragraph>

## References

- **<type>** `<ref>` — <description>
...

## Rules

1. <statement> (refs: <ref1>, <ref2>)
2. <statement>
...
```

See `assets/example-output.md` for a complete rendered example and `assets/example-input.json` for the JSON it was rendered from.

## Rendering rules

Apply each rule below deterministically. Do not improvise.

### Frontmatter

1. Emit frontmatter delimited by `---` on its own lines.
2. Emit `id:` and `layer:` verbatim from the input.
3. Emit `extends:` only when the input array is non-empty. Format as a YAML flow list: `extends: [parent-id-1, parent-id-2]`. Omit the field entirely when `extends` is missing, `null`, or `[]`.
4. Do not add any other frontmatter fields.

### Title

1. Derive the `# Title` from `id` by replacing hyphens with spaces and title-casing each word. Example: `py-import-order` → `# Py Import Order`.
2. If a word is a known technology acronym (`HTTP`, `API`, `SQL`, `JSON`, `YAML`, `URL`, `CLI`, `OS`, `TLS`, `JWT`, `CI`, `CD`, `UI`, `UX`, `SDK`, `RPC`, `DNS`), uppercase it.
3. If the id begins with a language prefix (`py-`, `go-`, `ts-`, `js-`, `java-`, `rust-`), render that prefix as `Python:`, `Go:`, `TypeScript:`, `JavaScript:`, `Java:`, `Rust:` followed by a space and the rest of the title. Example: `py-import-order` → `# Python: Import Order`.

### Purpose

1. Emit `## Purpose` followed by a blank line, then the `purpose` string verbatim.
2. Do not reflow, reformat, or truncate the purpose text.

### References section

1. Omit the `## References` heading entirely when the input `references` array is missing or empty — do not leave a bare heading.
2. When present, emit `## References` followed by a blank line, then one bullet per reference preserving input order.
3. Bullet format: `- **<type>** \`<ref>\` — <description>`. The `type` goes in bold; the `ref` goes in inline code backticks; separate from the description with a space, em-dash (`—`), space.
4. If `description` is missing, emit just `- **<type>** \`<ref>\`` with no em-dash.
5. Valid `type` values are `spec`, `adr`, `bdr`, `external`. Emit them in lowercase verbatim.

### Rules section

1. Always emit `## Rules` — rules are required by the schema.
2. Number rules starting at `1.` in input order.
3. Each rule is a single line: `<N>. <statement>`.
4. If the rule item has a non-empty `references` array, append ` (refs: <comma-separated refs>)` to the line. Example: `1. Group imports in three blocks. (refs: https://peps.python.org/pep-0008, ADR-014)`.
5. Do not wrap or split statements across multiple lines, even if long.

### Whitespace and structure

1. Exactly one blank line between the closing `---` of frontmatter and the `# Title`.
2. Exactly one blank line between each section heading and its content.
3. Exactly one blank line between sections.
4. End the file with a single trailing newline; no trailing blank lines beyond that.

## Workflow

1. **Validate the input.** Confirm `id`, `layer`, `purpose`, and `rules` are present. If `layer` is not one of the nine enum values, stop and report the invalid value — do not guess. If `rules` is empty, stop and report — a spec with no rules is malformed.
2. **Choose the output path.** Default to `specs/<layer>/<id>.md` relative to the repo root. If the user supplied a different path, use theirs.
3. **Render** by applying the rules above section-by-section. Assemble into a single string.
4. **Write** the file. If it already exists, confirm with the user before overwriting unless they explicitly said to overwrite.
5. **Report** the output path and a one-line summary of what was rendered (e.g. "Rendered py-import-order.md with 5 rules and 3 references").

## Input sources

Accept any of these input forms:

- JSON pasted inline in the user's message
- A path to a `.json` file — read it with the Read tool
- Multiple JSON objects in an array — render each to its own file under `specs/<layer>/<id>.md`

## Do not

- Do not invent content not present in the JSON.
- Do not reformat or normalize the `purpose` or rule `statement` prose.
- Do not add example sections, tables, footers, or metadata beyond what is specified.
- Do not emit empty optional sections as stub headings.
