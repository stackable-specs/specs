# Actor

The person, system, or organization performing an action. Actors are distinct from entities — an entity is acted upon, an actor acts. Actors carry roles and identity, which rules use for authorization, approval, and audit.

## Shape

```yaml
primitive: actor
name: <string>
roles:
  - <role>
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | The identifier for the actor in this context (e.g. `user`, `system`, `vendor`) |
| `roles` | list of strings | no | Role memberships used by authorization and approval rules |

## Use when

- A rule needs to know *who* is performing an action, not just *what* is being acted on
- Authorization decisions depend on the actor's role or identity
- Audit requirements capture the actor for traceability
- Approval workflows need to match an action to a permitted approver

## Composes with

- [command](../event/command.md) — a command is issued by an actor
- [approval-requirement](../governance/approval-requirement.md) — approver role is matched against actor roles
- [audit-requirement](../governance/audit-requirement.md) — actor identity is a required audit capture
- [boolean-decision](../decision/boolean-decision.md) — authorization decisions often resolve to allow/deny based on actor

## Example

```yaml
primitive: actor
name: user
roles:
  - FinanceManager
```

```yaml
primitive: actor
name: system
roles:
  - BillingAutomation
```
