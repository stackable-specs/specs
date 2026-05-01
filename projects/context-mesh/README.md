# **Project Specification: Context Mesh**

## 1. Executive Summary

### Overview

**Context Mesh** is a distributed, event-sourced organizational intelligence system that captures, interprets, and maintains shared context across teams using a combination of:

* Event ingestion (via n8n)
* Append-only event ledger (TimescaleDB)
* Local, user-bound context nodes (Go agents)
* AI-powered interpretation + human approval workflows

### Problem Being Solved

Organizations suffer from **context drift**:

* Decisions lose rationale
* Knowledge becomes siloed
* AI agents operate on inconsistent or outdated information
* Teams duplicate work or misalign on reality

### Proposed Solution

A **distributed context layer** where:

* All work events are captured
* AI agents interpret meaning
* Humans approve critical changes
* Canonical state is maintained with provenance
* Local agents provide personalized, permission-aware context

### Value Proposition

* Eliminates “what’s true?” ambiguity
* Makes decisions traceable and queryable
* Enables safe multi-agent AI collaboration
* Preserves permissions and organizational boundaries
* Reduces redundant work and misalignment

---

## 2. Objectives & Success Criteria

### Primary Objectives

1. Create a **single, append-only source of organizational truth**
2. Enable **AI-assisted interpretation with human governance**
3. Deliver **role-specific context to users automatically**
4. Maintain **strict permission boundaries across all derived data**

### Secondary Objectives

* Reduce meeting overhead
* Improve onboarding speed
* Increase decision transparency
* Enable multi-agent AI orchestration

### KPIs

| Metric                     | Target                            |
| -------------------------- | --------------------------------- |
| Decision traceability      | 100% of decisions have provenance |
| Context drift incidents    | ↓ 70%                             |
| Duplicate work detection   | ↑ 50%                             |
| Approval latency           | < 24 hours (avg)                  |
| Node sync accuracy         | 99.9%                             |
| Unauthorized data exposure | 0 incidents                       |

### Success Definition

* Users can answer: *“Why did this decision happen?”* in < 5 seconds
* Nodes remain consistent after offline periods
* No unauthorized data leaks across visibility scopes

### Failure Conditions

* Canonical state becomes untrusted
* Approval bottlenecks halt system flow
* Nodes diverge in interpretation significantly without resolution

---

## 3. Scope Definition

### In-Scope

* Event ingestion (Slack, meetings, GitHub, Drive)
* Append-only event storage
* AI interpretation layer
* Approval workflows (DRI-based)
* Canonical state projection
* Local context nodes (Go)
* Web UI per node
* Role-based permissions
* Multi-agent orchestration (basic)

### Out-of-Scope (Initial)

* Full enterprise identity federation
* Cross-company multi-tenant federation
* Autonomous decision-making without human approval
* Deep ML model training pipelines
* Real-time collaborative editing

### Assumptions

* Users have local machines capable of running Go agents
* External systems (Slack, Drive) remain source-of-truth for ACLs
* AI models are externally accessible (API-based)

---

## 4. User & Stakeholder Analysis

### Personas

#### 1. Builder (Engineer, IC)

* Needs: Accurate context, decisions, dependencies
* Pain: Misalignment, outdated specs
* Behavior: Consumes projections, occasionally contributes

#### 2. DRI (Manager / Owner)

* Needs: Approval clarity, decision accountability
* Pain: Lack of visibility into decisions
* Behavior: Reviews and approves changes

#### 3. Systems Steward

* Needs: System health, schema control
* Pain: Misclassification, governance complexity
* Behavior: Tunes system behavior

#### 4. Executive

* Needs: High-level insight, rationale
* Pain: Slow reporting chains
* Behavior: Queries system

#### 5. AI Agent User (Power User)

* Needs: Customize multiple AI agents
* Pain: Tool fragmentation
* Behavior: Configures agent roles and subscriptions

---

### Key Use Cases

* “Why did we delay launch?”
* “What changed since last week?”
* “What risks affect my project?”
* “What tools do I need access to?”
* “What did the AI steering committee decide?”

---

## 5. Functional Requirements

### Core Features

#### 1. Event Ingestion

* Input: external system events
* Output: raw_events
* Must deduplicate

#### 2. Event Interpretation

* Input: raw_events
* Output: interpreted_events
* Includes:

  * type
  * summary
  * confidence

#### 3. Quorum Interpretation (Advanced)

* Multiple interpreters process same event
* Results compared
* Discrepancies flagged

#### 4. Approval Workflow

* Interpreted events → approval queue
* DRI approves/rejects
* Output: approval_events

#### 5. Canonical State Projection

* Approved events → canonical tables
* Includes decisions, risks, dependencies

#### 6. Local Context Node

* Syncs from ledger
* Maintains local snapshot
* Enforces permissions
* Runs agent logic

#### 7. Web UI

* Node status
* Event inspection
* Approval interface
* Audit logs

#### 8. Multi-Agent Support

* Users assign roles to AI tools (e.g., Codex, Claude)
* Agents subscribe to event types

#### 9. Permission Enforcement

* Derived data respects source ACLs
* No expansion of visibility

---

### Edge Cases

* Conflicting interpretations → quorum resolution
* Missing permissions → access denied
* Node offline → replay events
* Duplicate approvals → idempotent handling

---

## 6. Non-Functional Requirements

### Performance

* Event ingestion latency: < 2s
* Query response: < 500ms

### Scalability

* Horizontal scaling via nodes
* TimescaleDB handles time-series partitioning

### Security

* Zero-trust model
* No shared credentials
* Local credential inheritance only

### Reliability

* Append-only ensures recoverability
* Node checkpointing prevents data loss

### Compliance

* GDPR/PII: enforced via visibility_scope
* Audit logs for all writes

---

## 7. System Architecture

### Overview

```text
External Systems → n8n → TimescaleDB → Context Nodes → Users
```

### Components

#### n8n

* Connectors
* Normalization

#### TimescaleDB

* Event ledger
* Canonical state

#### Context Nodes (Go)

* Sync engine
* AI agents
* UI server

---

### Data Flow

```text
Event → Raw → Interpreted → Approval → Canonical → Projection
```

---

### Tech Stack

| Layer     | Tech                          |
| --------- | ----------------------------- |
| Connector | n8n                           |
| DB        | TimescaleDB                   |
| Backend   | Go                            |
| UI        | React (optional lightweight)  |
| AI        | OpenAI / Claude / Gemini APIs |

---

## 8. Data Requirements

### Data Types

* Events (raw, interpreted)
* Decisions
* Risks
* Dependencies
* Nodes
* Visibility scopes

### Storage

* TimescaleDB hypertables
* JSONB payloads

### Privacy

* Visibility scope enforced at query time
* ACL intersection for derived data

---

## 9. Implementation Plan

### Phase 1: Event Ledger (2–3 weeks)

* TimescaleDB schema
* n8n ingestion

### Phase 2: Node Runtime (3–4 weeks)

* Go node
* Sync + checkpoint
* CLI

### Phase 3: Interpretation + Approval (4–6 weeks)

* AI integration
* DRI workflow

### Phase 4: Web UI (3–4 weeks)

* Dashboard
* Approvals
* Audit

### Phase 5: Multi-Agent + Quorum (4–6 weeks)

* Agent orchestration
* Discrepancy detection

---

### Team

* 1 Backend Engineer (Go)
* 1 Data Engineer (DB + n8n)
* 1 Frontend Engineer
* 1 AI Engineer
* 1 Product/Architect (you)

---

## 10. Risk Analysis

| Risk                 | Impact   | Mitigation             |
| -------------------- | -------- | ---------------------- |
| Approval bottleneck  | High     | thresholds + batching  |
| AI misclassification | Medium   | quorum + confidence    |
| Permission leaks     | Critical | strict ACL propagation |
| Node inconsistency   | Medium   | checkpoints            |
| User adoption        | Medium   | UI simplicity          |

---

## 11. Testing & Validation

### Testing Types

* Unit: event processing
* Integration: ingestion → canonical
* E2E: real scenarios (meetings, approvals)

### Acceptance

* All BDRs verified
* Manual queries return correct provenance

---

## 12. Deployment & Maintenance

### Deployment

* TimescaleDB: cloud or self-hosted
* n8n: containerized
* Nodes: local install

### Monitoring

* Node health
* Event lag
* Approval latency

### Maintenance

* Schema evolution via steward role
* Continuous iteration

---

## 13. Budget Considerations

### Cost Drivers

* DB hosting
* AI API usage
* Engineering time

### Optimization

* Cache interpretations
* Limit high-cost AI calls
* Batch processing

---

## 14. Open Questions

1. Optimal quorum size?
2. How to prevent DRI overload?
3. Schema evolution strategy?
4. Cross-team visibility boundaries?
5. Agent orchestration standard?

---

## Final Summary

You are building:

```text
A distributed, permission-aware, AI-augmented organizational memory system
```

Key innovation:

* **Local agents + central ledger**
* **AI quorum + human approval**
* **Strict ACL preservation**
* **Traceable, queryable decisions**
