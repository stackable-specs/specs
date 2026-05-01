# Workflow Spec: BDR Implementation Plan Verification

## Artifact Definition
- **Name:** BDR Implementation Plan Verification Report
- **Purpose:** Confirm that every gap claimed in a BDR implementation plan actually exists in the codebase before a developer writes a single line of implementation code. Surface plan inaccuracies, missing steps, and architectural risks before they become mid-implementation surprises.
- **Audience:** The developer assigned to implement the plan; the BDR author; any reviewer approving the plan before the implementation PR is opened.
- **Format:** Structured markdown report at `.tmp/bdr-NNN-verification-report.md`; one section per plan step; each section shows confirmed/wrong verdict with reproducible evidence. Concludes with a summary table and a list of issues not in the plan ranked by severity.

## Trigger
- **Type:** manual
- **Source:** Contributor invokes the workflow, supplying a BDR implementation plan path (e.g. `.tmp/bdr-002-implementation-plan.md`).
- **Condition:** The implementation plan document exists and lists discrete, file-level changes. The codebase is in its pre-implementation state (no partial work already merged).

## Inputs

| Source | Context |
|--------|---------|
| `.tmp/bdr-NNN-implementation-plan.md` | The plan — parse every claimed gap and the exact file path each gap refers to. |
| `.tmp/bdr-NNN-council-findings.md` | Original council decisions — used to verify the plan faithfully reflects what was settled, and to catch SQL/logic copied verbatim from findings that may contain errors. |
| Every source file named in the plan | Read each file in full to observe its current state before characterizing gaps. |
| Dependency manifests (`package.json`, `pnpm-workspace.yaml`, etc.) | Verify packages the plan says to install are not already present. |
| Framework/ORM config files | Identify runtime behaviors (e.g. `synchronize: true`, migration runner config) that affect whether a proposed change will behave as the plan claims. |

## Workflow Steps

1. **Parse the plan**
   - Read the implementation plan in full.
   - Extract: each numbered step, the file(s) it targets, and the specific change(s) claimed to be missing.
   - Build a checklist: one row per (step, file, claimed gap).
   - Halt if the plan has no file-level specifics — a plan that only describes intent without naming files cannot be verified at this level.

2. **Read every referenced file**
   - For each file named in the plan, read it in full.
   - Do not rely on summaries or memory — read the current content directly.
   - If a file does not exist, record `FILE MISSING` as the observation; do not infer what it would contain.

3. **Write and run static characterization checks**
   - Write a shell script at `.tmp/verification/bdr-NNN/static-checks.sh`.
   - For each claimed gap, write one `grep`-based check that produces a binary `CONFIRMED` (gap exists) or `WRONG` (claim is false) result.
   - Use self-relative paths (`ROOT="$(cd "$(dirname "$0")/../../../" && pwd)"`) so the script is portable.
   - Run the script and capture all output. Every claim must be resolved by a check — no unverified claims in the report.

4. **Write characterization tests**
   - For gaps that require runtime proof (e.g. a decorator not enforcing a constraint, a method accepting an input it should reject), write Jest characterization tests in `.tmp/verification/bdr-NNN/`.
   - Characterization tests prove **current behavior** — they must PASS now and FAIL once the plan is implemented.
   - Naming convention: `<domain>.characterize.ts` (e.g. `dto.characterize.ts`, `service.characterize.ts`, `schema.characterize.ts`).
   - Write a `jest.config.js` and `tsconfig.json` that resolve to the target app's `node_modules` via relative paths so tests run without installing new packages.
   - Use self-relative paths (`path.resolve(__dirname, '../../../apps/backend/src/...')`) inside test files.

5. **Run the characterization tests**
   - Execute `jest --config jest.config.js --no-coverage --verbose` from `.tmp/verification/bdr-NNN/`.
   - All tests must pass before the report is written — a failing characterization test means the spike itself is broken, not that the gap was verified.
   - Capture the full test output (suite names, test names, pass count, time).

6. **Cross-reference the plan against its source documents**
   - Re-read the council findings (or equivalent source document).
   - For each migration SQL block, schema definition, or algorithmic detail copied from the source into the plan, verify the copy is faithful and internally consistent.
   - Flag any case where the source document contains an error that the plan either propagates or silently corrects — both outcomes need to be noted.

7. **Identify issues not in the plan**
   - Look beyond the claimed gaps. For each proposed change, ask:
     - Does the runtime environment (ORM, framework, CI pipeline) need to be reconfigured to make this change take effect?
     - Does the change interact with existing data in a way that could fail on a non-empty database?
     - Does an equivalent utility already exist elsewhere in the codebase that the plan duplicates?
     - Does the plan's SQL or migration logic reference columns or types that do not yet exist at the time the SQL would run?
   - Classify each issue as `BLOCKER` (implementation will fail or be silently inert without this fix) or `RISK` (implementation will work in ideal conditions but will break under predictable real-world conditions).

8. **Compile the report**
   - Write `.tmp/bdr-NNN-verification-report.md` with the structure:
     - **Verdict:** one-sentence overall assessment (plan correct / plan has blockers).
     - **Gap Verification:** one subsection per plan step — confirmed/wrong verdict, evidence (grep result or test name), and any nuance.
     - **Critical Issues Not In The Plan:** one subsection per issue — severity (`BLOCKER` / `RISK` / `DESIGN NOTE`), finding, impact, and required amendment.
     - **Characterization Test Results:** full test run output (suite, test names, pass count, run command).
     - **Summary Table:** one row per plan step — status and short note.
   - Every claim in the report must trace to a grep result, a test name, or a direct file observation. No inferred verdicts.

9. **Update the verification report path in the plan (optional)**
   - If the implementation plan has a place to link the verification report, add the path. Do not modify the plan's steps or gap claims — those are the plan author's responsibility.

## Agent Configuration
- **Role:** Auditor. Reads the plan, reads the code, writes evidence-backed characterization tests, and produces a go/no-go report. Does not fix the plan, does not implement any plan steps, and does not modify production source files.
- **Trust Level:** `autonomous` — the report is written without requiring human approval at each step. The developer reviews the report before starting implementation.

## Validation Criteria
- [ ] Every step in the implementation plan has a corresponding subsection in the Gap Verification section.
- [ ] Every `CONFIRMED` verdict cites a specific grep match or passing characterization test — no verdict is asserted without evidence.
- [ ] Every `WRONG` verdict explains what was found instead of what the plan claimed.
- [ ] All characterization tests pass (`Tests: N passed, 0 failed`) before the report is finalized.
- [ ] The static-checks script runs end-to-end without errors from `.tmp/verification/bdr-NNN/`.
- [ ] The Issues Not In The Plan section covers at least: migration runner wiring, NOT NULL column safety with existing data, SQL column name consistency between source documents and the plan.
- [ ] No file reads are skipped — every file named in the plan was read directly, not inferred.
- [ ] The report's Summary Table covers every numbered plan step.
- [ ] The report is written to `.tmp/bdr-NNN-verification-report.md` (filename matches the BDR number).
- [ ] Characterization tests are isolated in `.tmp/verification/bdr-NNN/` with their own `jest.config.js` and `tsconfig.json`.

## Feedback Mechanism
- The developer reviews the report before opening the implementation PR. Any `BLOCKER` issue must be resolved in the plan before implementation begins — the developer updates the plan and re-triggers this workflow.
- `RISK` issues are addressed in the implementation itself (e.g. making a new NOT NULL column nullable initially) and documented in the PR description.
- `DESIGN NOTE` issues are documented in the PR description for reviewers to weigh in on.
- If a `WRONG` verdict is disputed (the plan author believes the claim is correct), the characterization test for that claim is the authoritative tie-breaker — it is examined together before implementation proceeds.

## Error Handling
- If a file named in the plan does not exist, record `FILE MISSING` in the Gap Verification section; do not infer what the file would contain or skip the check.
- If a characterization test fails to compile (not to assert), fix the test harness configuration before declaring any gap unverified — a test runner error is not evidence that the gap does not exist.
- If the council findings document is absent, skip Step 6 and note `source document unavailable — cross-reference skipped` in the report.
- If `synchronize: true` (or equivalent auto-migration behavior) is detected in the ORM config, always check whether proposed NOT NULL columns or type changes are safe with existing rows — this check is mandatory, not optional.

## Performance Targets
- Static checks complete in under 30 seconds.
- Characterization test suite runs in under 60 seconds.
- Full report is produced in a single agent session with no human input required.
- Zero characterization tests are left failing at the time the report is written.
