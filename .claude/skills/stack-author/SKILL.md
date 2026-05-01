---
name: stack-author
description: "Author or edit layered-spec markdown files in the stackable-specs repo (files under specs/LAYER/*.md). Use when creating a new spec, adding rules to an existing spec, refactoring a spec between layers, or reviewing a spec for schema conformance. Each spec has YAML frontmatter (id, layer, extends) and markdown sections for purpose, references, and a numbered list of imperative rules, matching references/schema.json. Layer enum values that should trigger this skill — language, platform, interface, presentation, delivery, observability, security, practices, quality."
---

# Spec Author

## Overview

Specs in this repo are markdown files that declare enforceable rules for one layer of a system. Every spec must conform to the JSON schema in `references/schema.json`. This skill converts that schema into a concrete markdown shape so specs stay machine-parseable and human-readable.

## Output shape

A spec is a single markdown file with YAML frontmatter for machine-readable fields and markdown body for prose fields. Copy `assets/spec-template.md` as the starting point.

```
---
id: <kebab-case-identifier>
layer: <one of the 9 layers>
extends: []
---

# <Human-readable title>

## Purpose

<One paragraph. Why this spec exists. What problem it prevents.>

## Do

- <Short positive-guidance bullet.>

## Don't

- <Short anti-pattern bullet.>

## References

- **spec** `<spec-id>` — short description
- **adr** `<ADR-###>` — short description
- **bdr** `<BDR-###>` — short description
- **external** `<url>` — short description

## Rules

1. <Imperative rule statement.> (refs: ADR-###, spec-id)
2. <Another rule.>
```

Omit optional sections entirely when empty — do not leave a bare `## References` heading with no bullets.

## Field semantics

### id (required)
Kebab-case, unique within the layer. Examples: `py-import-order`, `http-versioning`, `otel-trace-propagation`. The id must match the filename stem (`specs/language/py-import-order.md` → `id: py-import-order`).

### layer (required)
Exactly one of the values below. The `layer` field must match the directory the file lives in.

| Layer | Answers |
| --- | --- |
| `language` | What is it written in? |
| `platform` | Where does it run? |
| `interface` | How is it invoked? |
| `presentation` | What does it output? |
| `delivery` | How is it shipped? |
| `observability` | How is it inspected? |
| `security` | What must never break trust? |
| `practices` | How do we work? |
| `quality` | How do we enforce? |

If a candidate spec answers two layer questions, it is really two specs — split it.

### purpose (required)
A single paragraph under `## Purpose`. State *why* the spec exists, not *what* the rules are. The rules section carries the what.

### do / dont (optional)
Two parallel bulleted summaries — `## Do` for recommended practice, `## Don't` for the anti-patterns the spec prevents. They are scannable guidance; `## Rules` remains the canonical, citable contract.

- Each bullet is a short fragment (not a full numbered rule), modeled in the schema as a plain string.
- Do/Don't bullets distill the spirit of the Rules list — they should be derivable from `## Rules`, never contradict it.
- Both sections are optional. Omit either heading entirely when empty rather than leaving a stub.
- Authoring tip: write Do/Don't first as a thinking aid, then expand each bullet into one or more atomic rules below.

Schema fields are `do` and `dont` (both `array<string>`, both optional).

### extends (optional)
YAML list of spec ids that this one refines. Use when the current spec tightens or specializes a more general spec (e.g. a Python-specific refinement of a language-wide `import-order` spec). Set to `[]` (or omit the field) when empty.

### references (optional)
Supporting material. Four allowed types map to bullet prefixes:

| Schema `type` | Markdown prefix | `ref` contents |
| --- | --- | --- |
| `spec` | `- **spec** \`<id>\`` | Another spec's id in this repo |
| `adr` | `- **adr** \`<ADR-###>\`` | Architecture decision record identifier |
| `bdr` | `- **bdr** \`<BDR-###>\`` | Business decision record identifier |
| `external` | `- **external** \`<url>\`` | Any URL (RFC, blog post, vendor doc) |

Follow the ref with an em-dash and a short description.

### rules (required)
A numbered list of imperative statements under `## Rules`. Each rule:

- Is a single sentence the reader can comply with or violate.
- Is atomic — split conjunctions (`Use X and configure Y`) into separate rules.
- Is testable — a reviewer can check compliance without interpreting intent.
- May cite supporting references inline with `(refs: id1, id2)` at the end.

Do not embed rationale in the rule itself. Rationale belongs in `## Purpose` or a cited ADR/BDR.

## Workflow

1. **Pick the layer.** Read `/README.md` or `/specs/README.md` if unsure. If the spec answers two layer questions, split it.
2. **Check for an existing spec.** If one covers the topic, edit it rather than creating a parallel spec. Use `extends` to refine.
3. **Copy the template.** `cp assets/spec-template.md specs/<layer>/<id>.md` and fill it in.
4. **Validate before finishing:**
   - Frontmatter `layer` matches the directory.
   - Filename stem matches `id`.
   - Every rule is imperative, atomic, and testable.
   - Empty optional sections are omitted, not left as stub headings.

## Example

See `assets/example-spec.md` for a complete filled-in spec.

## Schema source of truth

The canonical schema lives at `references/schema.json`. If the schema changes, update this SKILL.md and the template/example to match — do not let them drift.
