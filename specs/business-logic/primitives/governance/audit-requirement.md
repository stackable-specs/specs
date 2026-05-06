# Audit Requirement

Specifies what must be captured for traceability whenever a rule fires. Audit requirements make rule outcomes reproducible and defensible by recording the actor, facts, logic, and result at the moment of evaluation — before any downstream mutation can alter the state.

## Shape

```yaml
primitive: audit_requirement
capture:
  - actor_id
  - timestamp
  - input_facts
  - matched_rules
  - outcome
  - reason_codes
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `capture` | string list | yes | Named fields that must be recorded in the audit log for each rule execution |
| `destination` | string | no | The log sink or storage system that should receive the audit record (e.g., `audit_db`, `s3://audit-bucket`) |
| `retention_policy` | reference | no | Reference to a `retention-requirement` primitive that governs how long this audit data must be kept |
| `on_failure` | enum | no | What to do if the audit write fails. One of: `block_outcome` (default), `warn_and_continue`, `silent_continue` |

## Capture field options

| Value | Description |
|-------|-------------|
| `actor_id` | Identity of the user or system that triggered the rule |
| `timestamp` | UTC instant at which the rule was evaluated |
| `input_facts` | The complete set of fact values passed into the rule |
| `matched_rules` | Which rules fired and which did not |
| `outcome` | The decision or effect that was produced |
| `reason_codes` | Machine-readable codes explaining the outcome |
| `policy_references` | The policy documents that authorized the rule |
| `overrides_applied` | Any manual overrides that modified the automated outcome |

## Use when

- A rule produces outcomes with legal, financial, or compliance implications
- You need to be able to reconstruct exactly what happened and why for a given decision
- Regulators, auditors, or customers may request a full trace of a specific decision
- The system must detect or prove tampering by recording immutable snapshots of decision inputs and outputs
- An approval or override workflow is in use and authorization must be documented

## Composes with

- [retention-requirement](./retention-requirement.md) — audit records have a defined lifecycle; retention requirement governs how long they must be kept before deletion or archival
- [approval-requirement](./approval-requirement.md) — approval events are part of the audit trail for any gated outcome
- [policy-reference](./policy-reference.md) — including policy references in the audit record makes each log entry self-contained for compliance purposes
- [../explanation/decision-trace](../explanation/decision-trace.md) — the decision trace is the primary artifact that an audit requirement causes to be persisted

## Example

```yaml
primitive: audit_requirement
capture:
  - actor_id
  - timestamp
  - input_facts
  - matched_rules
  - outcome
  - reason_codes
destination: audit_db
retention_policy: retain_7_years
on_failure: block_outcome
```

```yaml
primitive: audit_requirement
capture:
  - actor_id
  - timestamp
  - outcome
  - overrides_applied
  - policy_references
destination: s3://compliance-audit-bucket
on_failure: warn_and_continue
```
