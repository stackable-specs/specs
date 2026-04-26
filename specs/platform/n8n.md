---
id: n8n
layer: platform
extends: []
---

# n8n

## Purpose

n8n is a self-hosted workflow automation runtime that often ends up carrying business-critical integrations — invoicing, alerts, customer data sync — alongside production traffic. Its evaluation defaults (SQLite, no auth, in-memory queue, single process, generated encryption key) are explicitly unsafe for production: a restart can lose execution history, the editor becomes an unauthenticated remote-execution endpoint, a single runaway workflow can starve the queue, and a regenerated encryption key invalidates every stored credential. Storing workflows only inside the n8n database leaves no audit trail and no recovery path. This spec pins n8n's hosting, credential, workflow-authoring, and observability conventions so a team can run n8n in production with durable execution data, recoverable workflows under version control, authenticated access, and execution logs that flow into the same observability surface as the rest of the system.

## References

- **spec** `openobserve` — observability backend that consumes n8n logs and metrics
- **external** `https://github.com/n8n-io/n8n` — n8n source repository
- **external** `https://docs.n8n.io/` — n8n documentation
- **external** `https://docs.n8n.io/hosting/` — n8n self-hosting guide
- **external** `https://docs.n8n.io/hosting/configuration/environment-variables/` — environment variable reference
- **external** `https://docs.n8n.io/hosting/scaling/` — scaling and queue mode

## Rules

1. Pin the n8n version via Docker image tag (`n8nio/n8n:<X.Y.Z>`) or exact npm version; do not deploy from `latest`.
2. Run n8n against PostgreSQL as the persistent database in every non-local deployment; do not run production on the SQLite default.
3. Use Redis as the execution queue backend in production and run separate worker processes from the editor / main process.
4. Source `N8N_ENCRYPTION_KEY` from a secret manager and persist it across restarts; do not regenerate it during deploys, since rotation invalidates every stored credential.
5. Enable authentication on the editor UI in every non-local deployment (basic auth, SSO, or the owner-account login flow); do not expose the editor without auth.
6. Set `WEBHOOK_URL` to the externally reachable URL of the instance so generated webhook URLs route correctly.
7. Configure n8n through environment variables sourced from a secret manager or orchestrator config; do not bake credentials, tokens, or encryption keys into Docker images or committed compose files.
8. Export workflow JSON to a version-controlled repository and treat that as the source of truth; do not rely on the n8n database alone to preserve workflow definitions.
9. Name workflows in the form `[domain] verb-noun` (for example `[billing] sync-invoices`) so the workflow list is searchable.
10. Tag every workflow with its environment (`dev` / `staging` / `prod`), owner, and trigger type so the workflow list can be filtered without opening each workflow.
11. Define credentials only through n8n's Credential UI or `n8n credential:create` CLI, and reference them by name in workflows; do not embed secrets in node parameter expressions or hardcode them in JSON exports.
12. Wire every production workflow to an Error Trigger workflow that routes failures to the team's standard alerting channel.
13. Configure retry behavior on flaky external calls via the node's built-in retry settings; do not implement retry loops by chaining nodes manually.
14. Set an execution timeout on workflows that call external services so a runaway execution cannot starve the queue.
15. Extract reusable node sequences into sub-workflows invoked via the `Execute Workflow` node; do not copy-paste node sequences across workflows.
16. Use Code nodes only when n8n's expression language and built-in nodes cannot express the transformation; use the HTTP Request node for external I/O rather than `fetch` inside a Code node.
17. Enable n8n metrics (`N8N_METRICS=true`) and scrape them into the team's metrics store.
18. Forward n8n execution and audit logs to the central log store; do not let logs remain only inside the n8n container.
19. Configure execution-data pruning (`EXECUTIONS_DATA_PRUNE`, `EXECUTIONS_DATA_MAX_AGE`) with a documented retention window so the database does not grow unbounded.
20. Exercise workflow changes with a dedicated test-data sub-workflow or test trigger that injects representative inputs; do not rely on the production trigger to test development changes.
