# Workflow Spec: Stack Spec Lifecycle

## Artifact Definition
- **Name:** Stack Spec Lifecycle (add → ADR → implement → verify)
- **Purpose:** Take a layered spec from the central `specs/` library, adopt it into a target stack under `stacks/<stack>/`, justify the adoption with an ADR, implement the ADR's rules in the stack's code/config, and produce a verification artifact proving the implementation conforms.
- **Audience:** Stack owners, contributors adding capabilities to a stack, reviewers approving spec adoption, future readers reconstructing why the stack looks the way it does.
- **Format:** Four sequential artifact stages, each producing committed files in well-known locations (`stacks/<stack>/docs/specs/<layer>/<id>.md`, `stacks/<stack>/docs/adr/NNN-*.md`, code/config changes in the stack, `stacks/<stack>/docs/verify/<id>.md`).

## Trigger
- **Type:** manual
- **Source:** Contributor invokes the workflow when a new capability needs to land in a stack — typically `/stack-author add <thing>` followed by a request to "add to stack `<name>`".
- **Condition:** The spec being adopted exists (or is authored first) under `specs/<layer>/<id>.md` and conforms to `references/schema.json`.

## Inputs

| Source | Context |
|--------|---------|
| `specs/<layer>/<id>.md` | Canonical layered spec — purpose, references, numbered rules — to be adopted. |
| `specs/practices/madr.md` | MADR format the ADR must conform to (file naming, required sections, lifecycle). |
| `stacks/<stack>/docs/specs/` | Existing adopted specs — used to detect duplicates and infer stack-level conventions. |
| `stacks/<stack>/docs/adr/README.md` | ADR index — used to assign the next sequential `NNN`. |
| Stack codebase (`stacks/<stack>/...`) | Current implementation surface that ADR rules will modify or constrain. |
| Spec's referenced ADRs / sibling specs | Cross-stack dependencies the new ADR must cite (e.g. ADR-013 SBOM, ADR-002 uv). |

## Workflow Steps

1. **Stage 1 — Add spec to the stack**
   - Confirm the spec exists at `specs/<layer>/<id>.md`; if not, run `/stack-author` to author it first.
   - Verify the spec is not already present at `stacks/<stack>/docs/specs/<layer>/<id>.md`.
   - `mkdir -p stacks/<stack>/docs/specs/<layer>` and copy the spec file in unmodified — the stack consumes the central spec verbatim.
   - Read every `extends:` and `**spec**` reference in the file; if any cited spec is missing from the stack, surface it for adoption (loop) or document the gap.

2. **Stage 2 — Generate the ADR**
   - Read `stacks/<stack>/docs/adr/README.md` and the existing ADR filenames to assign the next sequential `NNN`.
   - Write `stacks/<stack>/docs/adr/NNN-adopt-<short-title>.md` conforming to MADR (Status / Context and Problem Statement / Decision Drivers / Considered Options / Decision Outcome / Consequences).
   - Frame the decision as the stack's adoption of the spec: cite the spec path, list realistic alternatives that were rejected, link related ADRs already in the stack.
   - Append the ADR to `stacks/<stack>/docs/adr/README.md` index.

3. **Stage 3 — Implement the ADR**
   - For each numbered rule in the adopted spec, identify the concrete change required in the stack's code/config (manifest, lockfile, Dockerfile, CI workflow, source files, etc.).
   - Make the edits. Prefer editing existing files over creating new ones. Each change should reference the ADR/spec rule it satisfies (in commit messages or PR description, not as code comments unless the *why* is non-obvious).
   - Where a rule is non-applicable to this stack, record the carve-out in the ADR's `## Consequences` section rather than ignoring it silently.
   - Run the stack's test, lint, and type-check gates locally; do not declare implementation complete on a red build.

4. **Stage 4 — Verify the ADR**
   - Produce `stacks/<stack>/docs/verify/<id>.md` (a verification report keyed to the spec id).
   - For every numbered rule in the spec, record one of: **Pass** (with file/line evidence), **N/A** (with the carve-out justification from the ADR's Consequences), or **Open** (with a tracked issue link and an owner).
   - Run any automated verification the stack provides (`trunk check`, `uv run pytest`, container scan, SBOM diff) and attach a digest of results.
   - If any rule is **Open**, the ADR's `## Status` stays `Proposed` until closure; flip to `Accepted` only when the verification report has zero **Open** rows.

## Agent Configuration
- **Role:** Multi-stage author and implementer. Stage 1 is a mechanical copy. Stage 2 is a structured-document author. Stage 3 is a code editor that translates rules to changes. Stage 4 is a structured verifier that maps rules to evidence.
- **Trust Level:** `review_required`. Each stage produces a discrete commit/PR that a human reviewer approves before the next stage starts. Stage 1 and 2 are low-risk; Stage 3 modifies stack code and must pass CI; Stage 4 is the gate that flips the ADR to `Accepted`.

## Validation Criteria
- [ ] Stage 1: spec file exists at `stacks/<stack>/docs/specs/<layer>/<id>.md` and is byte-identical to `specs/<layer>/<id>.md` (or has a documented divergence).
- [ ] Stage 1: every `extends:` and `**spec**` reference resolves to a spec already adopted in the stack, or is logged as a follow-up.
- [ ] Stage 2: ADR filename is `NNN-<kebab-case-title>.md`; `NNN` is the next unused sequence number; the file has all MADR-required sections.
- [ ] Stage 2: `## Considered Options` lists at least two alternatives plus the chosen option.
- [ ] Stage 2: ADR appears in `stacks/<stack>/docs/adr/README.md` with the correct title and `Accepted`/`Proposed` status.
- [ ] Stage 3: every implementation commit is linkable to one or more spec rules (commit message or PR body cites `spec/<id>` and rule numbers).
- [ ] Stage 3: stack-level CI is green at the end of the implementation stage (lint, type, unit, integration, security gates per the stack's spec selection).
- [ ] Stage 4: `stacks/<stack>/docs/verify/<id>.md` exists and covers every numbered rule in the source spec — no rule is silently omitted.
- [ ] Stage 4: each rule's status is one of `Pass | N/A | Open`; `Pass` rows cite file paths or command output; `N/A` rows link to the ADR Consequences carve-out; `Open` rows have an owner and a tracking link.
- [ ] Stage 4: ADR Status is `Accepted` only when no `Open` rows remain.
- [ ] No content is hallucinated: every cited file path, ADR number, and rule number resolves.

## Feedback Mechanism
- PR review on each stage's commit captures human signoff. Reviewers comment inline on the ADR or verification report; the author iterates before merge.
- If verification turns up an `Open` rule that cannot be closed, the ADR stays `Proposed` and a follow-up issue is filed, captured both in the verification report and in the stack's issue tracker.
- Quarterly: re-run Stage 4 verification across all `Accepted` ADRs in the stack to detect drift; any rule that flips from `Pass` to `Open` opens a remediation PR or supersedes the ADR.

## Error Handling
- Stage 1: if the source spec doesn't exist, halt and route to `/stack-author` instead of fabricating one.
- Stage 1: if the spec is already adopted with divergent content, halt and prompt the user — do not silently overwrite.
- Stage 2: if the next `NNN` is ambiguous (gap in numbering, parallel branches), halt and confirm with the user; never reuse a number.
- Stage 3: if implementing a rule requires a change outside the stack's scope (e.g. a new central spec, an org-wide policy), record it as a carve-out in the ADR's Consequences and an `Open` row in verification — do not silently skip.
- Stage 4: if automated verification tools are unavailable, mark the affected rows `Open — unverified` rather than `Pass`.

## Performance Targets
- Stage 1: under 5 minutes (mechanical copy + reference resolution).
- Stage 2: under 30 minutes for a single ADR; produces a draft a reviewer can approve in one pass.
- Stage 3: bounded by the spec's rule count and the stack's CI runtime; typical adoption ≤ 1 day for an existing stack.
- Stage 4: 90% of verification reports pass review without revision; 100% of `Accepted` ADRs have a corresponding verify report.
