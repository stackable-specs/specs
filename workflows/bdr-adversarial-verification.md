# Workflow Spec: BDR Adversarial Verification

## Artifact Definition
- **Name:** BDR Adversarial Verification Report
- **Purpose:** Confirm that a feature's externally observable behavior matches its BDR contract, then attempt to break it via adversarial inputs, and reconcile both results with live traces from OpenObserve.
- **Audience:** Feature authors, QA engineers, and reviewers deciding whether to flip a BDR's status from `Proposed` to `Accepted` (or flag a regression in an `Accepted` BDR).
- **Format:** Structured markdown report in `.tmp/bdr-NNN-verification-report.md`; one section per BDR scenario; each section shows assertion outcome, adversarial findings, and trace/metric evidence. Concludes with a pass/fail verdict and recommended status transition.

## Trigger
- **Type:** manual
- **Source:** Contributor invokes the workflow, supplying a BDR file path (e.g. `docs/bdr/001-user-registration.md`).
- **Condition:** The target application is running and reachable; OpenObserve is available and receiving traces/metrics from the application. If the application is not running or has uncommitted code changes that have not been loaded, restart it before proceeding (see Step 0).

## Inputs

| Source | Context |
|--------|---------|
| `docs/bdr/NNN-<title>.md` | The BDR file — parse `## Behavior`, `## Acceptance Criteria`, and `## Verification` sections to derive test scenarios. |
| Application API base URL | Used by curl-based subagents for all API interactions (no SDK or internal test harness). |
| Application frontend URL | Used by playwright-cli subagents for UI interactions. |
| OpenObserve endpoint + credentials | Used by the openobserve skill to query traces and metrics after each test phase. |
| `docs/bdr/index.md` | BDR index — used to record the final verification outcome against the correct entry. |

## Workflow Steps

0. **Restart the application and resolve the project `.tmp` path**
   - Determine the project root from the BDR file path (e.g. `docs/bdr/002-team-creation.md` → project root is the directory containing `docs/`). All report output goes to `<project-root>/.tmp/`.
   - Check whether the running application reflects the current source code. A reliable signal is comparing the error messages or response shapes from a known endpoint against what the current source produces. If the application is stale (e.g. source was changed but server not restarted), restart it now before running any assertions. Do not begin the assertion phase against a stale process — stale-server failures produce misleading reports.
   - After restart, confirm the application is reachable (HTTP 200 on a health or docs endpoint) before proceeding.

1. **Parse the BDR**
   - Read the specified BDR file.
   - Extract: `## Behavior` (one-sentence capability), all `## Acceptance Criteria` items (AC-1…AC-N), and all `## Verification` scenarios (Given/When/Then).
   - Classify each scenario as API-testable (curl) or UI-testable (playwright-cli) based on whether the Then clause references HTTP responses/status codes or a UI state.
   - Halt if the BDR status is `Rejected` or `Deprecated`; do not verify a retired contract.

2. **Assertion phase — conformance subagents**
   - Launch one subagent per BDR scenario in parallel.
   - Each subagent:
     - Uses **only `curl`** (for API scenarios) or **only the `playwright-cli` skill** (for UI scenarios) — no internal test harness, no SDK calls.
     - Executes the Given→When sequence and captures the raw response.
     - Evaluates the Then clause against the captured response.
     - Records: scenario ID, tool used, request sent, response received, pass/fail verdict, and the exact Then clause checked.
   - Collect all subagent results before proceeding.

3. **Adversarial phase — break-it subagents**
   - Launch a team of adversarial subagents (minimum three, one per attack class below) in parallel:
     - **Boundary probe:** submit inputs at and beyond stated limits (e.g. minimum password length - 1, maximum field length + 1, empty string, null, Unicode edge cases).
     - **Duplicate/idempotency probe:** replay the exact same valid request twice; verify the system handles the repeat correctly (no duplicate records, correct error, idempotent response).
     - **Authorization probe:** attempt to trigger the behavior as an unauthorized actor (no credentials, expired token, wrong role); verify the system rejects correctly without leaking data.
     - **Malformed input probe:** send structurally invalid payloads (missing required fields, wrong content-type, oversized body, SQL/script injection strings in text fields).
   - Each adversarial subagent uses **only `curl`** or **only `playwright-cli`** — same constraint as the assertion phase.
   - Each subagent records: attack class, payload sent, response received, and whether the system behaved as the BDR implies it should (or found an unexpected gap).

4. **Observability verification — OpenObserve**
   - After both assertion and adversarial phases complete, use **only the `openobserve` skill** to:
     - Query traces for the time window covering both phases; confirm a trace exists for each BDR scenario exercised.
     - Confirm that successful scenarios produced HTTP 2xx spans and failed/rejected scenarios produced 4xx spans (no silent 5xx on expected-failure paths).
     - Pull request-rate and error-rate metrics for the endpoints touched; flag any anomalous spike that the BDR does not account for.
     - Capture span attributes (endpoint, status code, duration) as evidence for each scenario.
   - Record the OpenObserve query used and the result for each scenario in the report.

5. **Compile the report**
   - Resolve the output path as `<project-root>/.tmp/bdr-NNN-verification-report.md`, where `<project-root>` is the directory containing the BDR file's `docs/` folder. Example: if the BDR is at `/path/to/project/docs/bdr/002-team-creation.md`, write the report to `/path/to/project/.tmp/bdr-002-verification-report.md`.
   - Write `<project-root>/.tmp/bdr-NNN-verification-report.md` with the structure:
     - **Header:** BDR number, title, behavior statement, report timestamp.
     - **Assertion Results:** one row per scenario — scenario ID, tool, pass/fail, evidence (request + response excerpt).
     - **Adversarial Findings:** one row per attack class — attack description, payload, response, verdict (contained / gap found).
     - **Observability Evidence:** one block per scenario — OpenObserve query, trace count, span status distribution, metric snapshot.
     - **Verdict:** overall pass/fail; list any acceptance criteria not fully confirmed.
     - **Recommended Status Transition:** `Proposed → Accepted` (all AC confirmed, no gaps), `Proposed → Proposed` (open items remain, list them), or `Accepted → regression` (existing BDR has a new failure).
   - Do not hallucinate results — every claim in the report must trace to a captured response or an OpenObserve query result.

6. **Update the BDR index**
   - Append or update the entry in `<project-root>/docs/bdr/index.md` with the verification date and the recommended status.
   - Do not modify the BDR file itself — status transitions require a separate PR per the BDR spec rules.

## Agent Configuration
- **Role:** Orchestrator + evidence collector. The orchestrator parses the BDR and dispatches parallel subagents. Subagents are strictly black-box testers — they have no access to source code or internal test frameworks. The orchestrator synthesizes subagent results and the OpenObserve data into the final report.
- **Trust Level:** `review_required` — the report is written automatically, but a human reviewer confirms the recommended status transition before the BDR file is updated.

## Validation Criteria
- [ ] The application was confirmed running the current source code before assertions began (Step 0 completed; server restarted if stale).
- [ ] Every `## Acceptance Criteria` item from the BDR has at least one corresponding assertion result (pass or fail) in the report.
- [ ] Every `## Verification` scenario has been executed by exactly one subagent using only `curl` or `playwright-cli`.
- [ ] All four adversarial attack classes (boundary, duplicate/idempotency, authorization, malformed input) have been attempted and documented.
- [ ] Each report row cites the raw request sent and the raw response or UI state observed — no inferred or paraphrased results.
- [ ] OpenObserve traces exist for the time window of each test phase and are cited in the report.
- [ ] No 5xx responses appear on expected-failure paths (adversarial inputs should produce 4xx, not server errors).
- [ ] The report is written to `<project-root>/.tmp/bdr-NNN-verification-report.md` (path is relative to the project being tested, not the workflow file).
- [ ] The `<project-root>/docs/bdr/index.md` entry for the BDR reflects the verification date and recommended status.
- [ ] No claims in the report are unsupported by captured evidence — zero hallucinated pass results.
- [ ] The verdict is binary: all acceptance criteria confirmed → `Accepted`; any open item → `Proposed` with a list.

## Feedback Mechanism
- The report is reviewed in the PR that proposes the BDR status transition. Reviewers comment on specific scenario rows; the author re-runs the relevant subagent and updates the report before merge.
- If an adversarial probe reveals a gap not covered by any acceptance criterion, a follow-up BDR is drafted to capture the new contract.
- After the BDR is accepted, re-run this workflow on each PR that modifies the feature's code paths; any regression flips the BDR entry in the index to `regression` and blocks merge until resolved.

## Error Handling
- If the application is not reachable, restart it (Step 0) and wait for the health check to pass. If still unreachable after restart, halt and log `application-unavailable` — do not fabricate responses.
- If the application is reachable but error messages or response shapes indicate stale code (e.g. error text does not match what the current source would produce), halt, restart the server, confirm the health check, and restart the workflow from Step 1.
- If OpenObserve returns no traces for the test window, mark observability rows `Open — no traces` rather than `Pass`; the BDR cannot be accepted without trace evidence.
- If a subagent times out or errors, mark the corresponding scenario `Open — subagent failure` and include the error message in the report.
- If the BDR file has no `## Verification` section, halt and report `malformed-bdr` — do not infer scenarios from the Acceptance Criteria alone.
- If the BDR status is already `Accepted` and all assertions pass, the report verdict is `Accepted — no regression` and no index update is needed.

## Performance Targets
- Assertion and adversarial phases run in parallel; total elapsed time should be under 10 minutes for a BDR with ≤ 5 scenarios.
- Report compilation completes within 2 minutes of all subagent results being collected.
- 100% of BDRs transitioned to `Accepted` have a corresponding verification report in `.tmp/`.
- Adversarial phase finds zero unhandled 5xx responses on any expected-failure path across all accepted BDRs.
