# Workflow Spec: BDR Council Review

## Artifact Definition
- **Name:** BDR Council Findings Report
- **Purpose:** Resolve open items and adversarial gaps from a BDR verification report through structured multi-persona debate, producing settled implementation decisions and a clean list of questions that require human stakeholder input.
- **Audience:** Feature engineers implementing the fixes, product owners deciding on terminology and scope, and reviewers approving the BDR status transition to `Accepted`.
- **Format:** Structured markdown report in `.tmp/bdr-NNN-council-findings.md`; one section per open item; each section contains the final settled decision, layer-by-layer implementation steps, and migration SQL where applicable. Concludes with an explicit **Unanswerable Questions** section listing only what requires human input.

## Trigger
- **Type:** manual
- **Source:** Contributor invokes the workflow after a BDR verification report is written with a status of `Proposed` and one or more open items blocking `Accepted`.
- **Condition:** A `.tmp/bdr-NNN-verification-report.md` exists for the target BDR with at least one open item in the **Verdict** section.

## Inputs

| Source | Context |
|--------|---------|
| `.tmp/bdr-NNN-verification-report.md` | The verification report — read the **Verdict** section to extract the numbered open items blocking Accepted status, plus any adversarial findings (boundary, authorization, XSS, idempotency gaps). |
| `docs/bdr/NNN-<title>.md` | The BDR file — used to understand existing acceptance criteria, the feature's behavior contract, and any referenced sibling BDRs. |
| `projects/<app>/` | Application source — research subagents read the codebase to answer each open item with grounded evidence (schema, DTOs, service patterns, frontend types). |

## Workflow Steps

### Phase 1 — Formulate Questions

1. **Read the verification report**
   - Extract all open items from the **Verdict** section.
   - Extract all adversarial findings marked as **gap** from the adversarial probe tables.
   - For each item, identify the category: validation constraint, security/authorization, XSS/sanitization, data uniqueness/idempotency, UI/rendering gap.

2. **Generate follow-up questions**
   - For each open item, produce one or more targeted questions whose answers will unlock an implementation decision.
   - Questions must distinguish between: product decisions (what the rule should be), implementation decisions (how to enforce it), and unknowns (what the current codebase already does).

### Phase 2 — Parallel Research

3. **Launch one research subagent per open item**
   - Run all research subagents in parallel.
   - Each subagent reads only the codebase files relevant to its question: schema/entities, DTOs, services, controllers, frontend components, BDR documents, ADRs.
   - Each subagent reports: current state (what exists now), any existing patterns the fix should follow, and the specific gap between current state and the required behavior.
   - Research subagents do **not** write code or make recommendations — they surface facts only.

### Phase 3 — Council Debate (3 Rounds)

4. **Dynamically select council personas** based on the nature of the open items:
   - Always include a **Security Architect** if any authorization, IDOR, XSS, or injection gap was found.
   - Always include a **Backend API Engineer** if any DTO, validation, or service-layer gap was found.
   - Always include a **Database Architect** if any schema constraint, uniqueness, or migration gap was found.
   - Always include a **Frontend Engineer** if any UI rendering, TypeScript type, or form validation gap was found.
   - Always include a **Product Owner** if any open item requires an explicit acceptance criterion or BDR update.
   - Minimum council size: 3 personas. Maximum: 5.

5. **Round 1 — Initial positions**
   - Launch all council persona subagents in parallel.
   - Each persona receives: the full research findings from Phase 2, their role and expertise, and all 6 open items.
   - Each persona stakes a clear, opinionated position on every open item from their domain perspective.
   - No hedging — each persona makes a call and defends it.

6. **Round 2 — Cross-examination**
   - Compile all Round 1 positions into a single shared context.
   - Identify the contested points (positions where at least two personas disagree).
   - Launch all council persona subagents in parallel, each receiving their Round 1 position plus all other personas' Round 1 positions.
   - Each persona challenges disagreements, defends their position, and explicitly states if another argument changed their mind (and why).

7. **Round 3 — Final positions**
   - Compile all Round 2 positions into a single shared context.
   - Identify any items still unresolved after Round 2.
   - Launch all council persona subagents in parallel for their final, definitive recommendations.
   - Each persona must: (a) state the final settled decision with implementation steps, and (b) flag with `[HUMAN INPUT REQUIRED]` anything that cannot be resolved without stakeholder input.

### Phase 4 — Synthesis and Output

8. **Synthesize the council output**
   - For each open item:
     - If all personas agree (or moved to agreement by Round 3): mark **SETTLED** and compile implementation steps across all layers.
     - If a majority agrees but one dissents: record the majority position, note the dissent and its reasoning, and mark **SETTLED** with a note.
     - If genuine disagreement persists: mark **[HUMAN INPUT REQUIRED]** and state precisely what decision is needed.
   - Collect all `[HUMAN INPUT REQUIRED]` flags from all personas into a single **Unanswerable Questions** section at the end of the report.

9. **Write the findings report**
   - Write `.tmp/bdr-NNN-council-findings.md` with the structure:
     - **Header:** BDR number, title, date, council composition, number of rounds.
     - **One section per open item:** status (SETTLED / HUMAN INPUT REQUIRED), final decision, implementation steps by layer (DB, DTO, service, controller, frontend, BDR AC text).
     - **SQL migration blocks** for any schema changes, including deduplication steps for uniqueness constraints.
     - **Unanswerable Questions:** numbered list; each question states what is unknown, why it cannot be resolved by engineering, and what work is blocked until it is answered.

## Agent Configuration
- **Role:** Orchestrator + debate facilitator. The orchestrator formulates questions, dispatches research subagents, selects council personas, runs 3 debate rounds, and synthesizes the final report. Council persona subagents are domain experts with strong opinions — they do not read the codebase directly; they reason from the research findings fed to them.
- **Trust Level:** `review_required` — the findings report is written automatically, but a human must answer the Unanswerable Questions before implementation begins.

## Validation Criteria
- [ ] Every open item from the verification report's **Verdict** section has a corresponding section in the findings report.
- [ ] Every settled decision includes implementation steps at every relevant layer (DB, DTO, service, frontend, BDR).
- [ ] Every schema change includes exact migration SQL with a deduplication or pre-check step where applicable.
- [ ] The Unanswerable Questions section contains only questions that genuinely cannot be resolved from the codebase, BDRs, or existing ADRs — no engineering questions that research subagents could have answered.
- [ ] No settled decision contradicts evidence from the research phase — all decisions are grounded in codebase facts.
- [ ] Round 2 identifies every contested point from Round 1 and feeds it explicitly to each persona.
- [ ] Round 3 produces a definitive call on every item — no persona ends Round 3 with "it depends" without specifying what it depends on.
- [ ] Security findings (IDOR, XSS, injection) are never marked `[HUMAN INPUT REQUIRED]` for the fix itself — only for label/scope decisions around the fix.
- [ ] The findings report is written to `.tmp/bdr-NNN-council-findings.md` matching the BDR number.

## Feedback Mechanism
- The findings report is reviewed alongside the implementation PR. Reviewers verify that each implementation matches the settled decision.
- Unanswerable Questions are resolved by the product owner as comments on the findings file or in the PR description before merge.
- After all questions are answered and implementation ships, re-run the `bdr-adversarial-verification` workflow to confirm the open items are closed and the BDR can transition to `Accepted`.
- If a Round 3 dissent was overruled and the majority decision later proves incorrect, the dissenting position and its reasoning are preserved in the council findings for retrospective learning.

## Error Handling
- If the verification report has no **Verdict** section or no open items, halt and report `no-open-items` — this workflow is not needed.
- If a research subagent returns no codebase evidence for an open item (file not found, pattern not present), mark that item's research as `inconclusive` and treat it as requiring human input rather than guessing.
- If a council persona produces a position that contradicts a research finding, flag the contradiction explicitly in the synthesis — do not silently resolve it.
- If fewer than 3 round-trip exchanges occur (i.e., a persona skips a round), the synthesis must note the incomplete debate and flag affected findings as lower-confidence.
- If all 5 personas reach full consensus in Round 1 on a finding, skip Rounds 2 and 3 for that finding only — do not run unnecessary debate rounds.

## Performance Targets
- Research phase (parallel subagents): under 3 minutes for up to 6 open items.
- Each council round (parallel subagents): under 2 minutes per round.
- Total elapsed time for 3 rounds + synthesis: under 15 minutes.
- Unanswerable Questions list contains the minimum number of questions possible — every question that can be answered by the codebase or existing BDRs should be answered by research, not escalated.
- 100% of settled findings result in implementations that pass re-verification on the `bdr-adversarial-verification` workflow.
