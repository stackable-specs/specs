# BLT Practices

Guidance for using the Business Logic Template primitive catalog to author, compose, and maintain business logic specs.

See [BLT.md](../BLT.md) for design rationale and [primitives/](../primitives/README.md) for the full catalog.

---

## What a BLT spec is

A BLT spec is a composed document that expresses a business rule or process using named primitives from the catalog. It is not code. It is not a free-form description. It is a structured, testable, auditable declaration of intent — close enough to implementation that an engineer can build it directly, and close enough to policy that a business analyst can validate it.

The unit of authoring is the **rule**. A rule is always:

```
WHEN  <trigger>
GIVEN <facts>
IF    <conditions>
THEN  <effects>
```

Complex processes are compositions of rules.

---

## Authoring workflow

Follow this sequence when authoring a new BLT spec:

### 1. Name the trigger

Every rule starts with what causes it to fire. Before writing anything else, identify the trigger type and name.

```yaml
trigger:
  type: event | command | schedule | state_change | manual
  name: PaymentFailed
```

If you cannot name the trigger, the rule is not ready to be authored.

### 2. Identify the primary entity

What business object does the rule act on? Name it explicitly. This forces you to resolve ambiguity early — "the customer" and "the account" are different entities.

```yaml
entity: Invoice
```

### 3. Declare the facts

List every input the rule needs. Distinguish sourced facts from derived facts. Mark each as required or optional. For any required fact, declare what happens when it is missing.

```yaml
facts:
  - name: invoice.amount_due
    type: money
    source: billing_service
    required: true

  - name: invoice.due_date
    type: date
    source: billing_service
    required: true

derived_facts:
  - name: invoice.days_past_due
    derive:
      expression: days_between(invoice.due_date, today)
```

### 4. Write the conditions

Use the condition primitives. Prefer named predicates over inline expressions when a condition will be reused or when its business meaning should be explicit.

```yaml
conditions:
  all:
    - invoice.amount_due > 0
    - invoice.days_past_due > 30
    - account.in_collections = false
```

Avoid burying logic in field names. `invoice.is_overdue` is a derived fact or predicate, not a field. Name the thing, define it once, reference it everywhere.

### 5. Define the effects

Use effect primitives. One rule can have multiple effects. Order matters only within side effects that depend on each other — use a pipeline for sequenced effects, not inline ordering.

```yaml
effects:
  - create_obligation:
      type: CollectionFollowUp
      owner: CollectionsQueue
      due_at: today + 2 business_days

  - notify:
      recipient: account.primary_contact_email
      channel: email
      template: overdue_notice
```

### 6. Declare exceptions

Exceptions are not edge cases to handle later. They are anticipated deviations that belong in the spec from the start.

```yaml
exceptions:
  - name: StrategicAccountOverride
    applies_when:
      - customer.tier = Strategic
    effect:
      route_to: AccountManagerReview
```

### 7. Add reason codes

Every decision and every blocked action needs a reason code. Reason codes are the contract between the rule and everything downstream — reporting, notifications, customer-facing messages, audits.

```yaml
reason_codes:
  - OverdueBalance
  - StrategicAccountOverride
```

### 8. Specify audit capture

Declare what must be recorded when the rule fires. At minimum: actor, timestamp, input facts, outcome, reason codes.

```yaml
audit:
  capture:
    - actor_id
    - timestamp
    - invoice.amount_due
    - invoice.days_past_due
    - outcome
    - reason_codes
```

---

## Composition patterns

Most business processes are not a single rule. Use these composition primitives to combine rules:

| Pattern | Use when |
|---------|----------|
| [Rule Set](../primitives/composition/rule-set.md) | Multiple independent rules apply to the same entity; collect all results |
| [Pipeline](../primitives/composition/pipeline.md) | Steps must execute in order and each feeds the next |
| [Branch](../primitives/composition/branch.md) | The process forks based on a condition |
| [Gate](../primitives/composition/gate.md) | A human decision or external signal must be received before proceeding |
| [Join](../primitives/composition/join.md) | Parallel tracks must all complete before continuing |
| [Decision Table](../primitives/composition/decision-table.md) | Many input combinations map to specific outputs |

### Example: eligibility + calculation + approval

```yaml
spec:
  id: refund_request_v1
  type: rule_set
  evaluation_mode: collect_all

rules:
  - id: refund_eligibility
    # ... eligibility rule

  - id: refund_amount_calculation
    # ... calculation rule

  - id: refund_approval_gate
    if:
      - refund.amount > 500
    then:
      - create_obligation:
          type: RefundApproval
          owner: FinanceManager
```

---

## Naming conventions

| Thing | Convention | Example |
|-------|-----------|---------|
| Spec ID | `snake_case_v{n}` | `refund_eligibility_v2` |
| Entity | PascalCase | `Invoice`, `Subscription` |
| Fact | `entity.field_name` | `invoice.days_past_due` |
| Event | PascalCase past tense | `PaymentFailed`, `OrderShipped` |
| Command | PascalCase imperative | `SubmitInvoice`, `RequestRefund` |
| Reason code | PascalCase noun phrase | `OutsideReturnWindow`, `CreditHoldActive` |
| Obligation type | PascalCase noun | `CollectionFollowUp`, `RefundApproval` |
| Template ID | `snake_case` | `payment_failed_notice` |

---

## What belongs in a BLT spec vs. elsewhere

| Belongs in a BLT spec | Belongs elsewhere |
|-----------------------|-------------------|
| Business rule logic | Implementation code |
| Trigger and fact declarations | Database schema |
| Conditions and outcomes | UI presentation |
| Reason codes | Localized message strings |
| Audit requirements | Logging infrastructure config |
| Exception handling | Error monitoring setup |
| Test cases (given/expect) | Integration test suites |

BLT specs are policy artifacts. They should be readable by engineers, product managers, and business analysts without translation.

---

## Testing

Every spec should include at minimum:

- A **happy path** case: all conditions met, expected outcome produced
- A **negative case**: one condition fails, correct reason code returned
- An **exception case**: exception applies, override outcome produced
- A **missing fact case**: required fact absent, fallback triggered

```yaml
tests:
  - name: eligible_refund
    given:
      order.status: Delivered
      order.delivered_at: 2026-04-20
      order.final_sale: false
      payment.chargeback_open: false
    expect:
      refund_eligibility: Eligible
      reason_codes: [RefundPolicySatisfied]

  - name: outside_return_window
    given:
      order.status: Delivered
      order.delivered_at: 2026-03-01
      order.final_sale: false
    expect:
      refund_eligibility: Ineligible
      reason_codes: [OutsideReturnWindow]

  - name: missing_delivery_date
    given:
      order.status: Delivered
      order.delivered_at: null
    expect:
      refund_eligibility: NeedsReview
      reason_codes: [DeliveryNotConfirmed]
```

---

## Versioning

- Increment the version number in the spec ID when the rule logic changes in any way that affects outcomes for existing records.
- Use [Rule Version](../primitives/governance/rule-version.md) to declare `effective_from` so as-of evaluation works correctly.
- Use [Rollout Control](../primitives/governance/rollout-control.md) to release changes gradually.
- Use [Kill Switch](../primitives/governance/kill-switch.md) to revert without a deployment.

Breaking changes (outcome values change, new required facts, reason codes removed) always require a version bump. Non-breaking changes (new optional facts, new reason codes added, description updates) may be made in place.

---

## Common mistakes

**Burying logic in derived facts.** A derived fact like `invoice.should_send_to_collections` is a decision masquerading as a fact. Use a [Boolean Decision](../primitives/decision/boolean-decision.md) instead.

**Omitting reason codes.** A rule that produces an outcome without a reason code cannot be explained, reported on, or debugged. Every outcome path needs at least one reason code.

**Skipping the missing-fact case.** If a fact is required and could be absent in production, the spec is incomplete without a [Missing Fact](../primitives/fact/missing-fact.md) declaration.

**One giant rule.** If a rule has more than five conditions and three effects, it is probably two or three rules composed together. Split it and use a rule set or pipeline.

**Hardcoding thresholds inline.** A threshold like `refund.amount > 500` that appears in multiple rules should be a named [Threshold](../primitives/condition/threshold.md) primitive, not repeated inline.

**Conflating events and commands.** An event is something that happened (past tense, immutable). A command is a request (imperative, may be rejected). Use the right trigger type — it affects idempotency, error handling, and audit semantics.
