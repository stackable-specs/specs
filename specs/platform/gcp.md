---
id: gcp
layer: platform
extends: []
---

# Google Cloud Platform

## Purpose

Google Cloud is a multi-tenant substrate where one mis-scoped IAM role, one downloaded service-account key, one publicly addressable Cloud SQL instance, or one project pinned to `roles/owner` for "convenience" can hand an attacker the keys to the whole estate. The defaults that make GCP easy to start with — basic roles, console-driven changes, automatically created default service accounts, public IP allocation, and unlabelled spend — are exactly the defaults that make a production GCP environment ungovernable: no clean blast-radius boundary between workloads, no audit trail showing who changed what, no cost attribution, no way to rotate or revoke long-lived credentials, and no enforcement that prevents a tired engineer from re-creating yesterday's mistake. This spec pins the resource hierarchy, identity and credential model, network defaults, secret handling, observability sinks, and change-management gates so a GCP-hosted workload has a defensible, auditable, cost-attributable home — not a sprawl of click-ops resources spread across a billing account no one fully reads.

## References

- **spec** `terraform` — sibling delivery-layer spec for the IaC that provisions GCP
- **spec** `github-actions` — sibling delivery-layer spec for CI that authenticates to GCP via OIDC
- **external** `https://cloud.google.com/` — Google Cloud home
- **external** `https://cloud.google.com/architecture/framework` — Google Cloud Architecture Framework
- **external** `https://cloud.google.com/iam/docs/best-practices-for-managing-service-account-keys` — Service account key best practices
- **external** `https://cloud.google.com/iam/docs/workload-identity-federation` — Workload Identity Federation
- **external** `https://cloud.google.com/resource-manager/docs/organization-policy/overview` — Organization Policy Service
- **external** `https://cloud.google.com/security-command-center/docs/security-command-center-overview` — Security Command Center
- **external** `https://cloud.google.com/logging/docs/audit` — Cloud Audit Logs

## Rules

1. Organize resources under a Google Cloud Organization with a folder hierarchy that separates environments (`prod`, `nonprod`) and business units; do not run workloads under a standalone project that is not attached to an organization.
2. Provision one project per workload-environment pair (e.g. `<service>-prod`, `<service>-staging`); do not co-locate unrelated workloads or multiple environments in a single project.
3. Provision and modify resources through Terraform (or another IaC tool); do not make changes to production resources from the Cloud Console or ad-hoc `gcloud` commands.
4. Grant IAM roles to Google Groups, not individual user principals, for any human-accessible role; do not add named users directly to resource IAM policies.
5. Do not grant the basic roles `roles/owner`, `roles/editor`, or `roles/viewer` on any project, folder, or organization that hosts production workloads — use predefined or custom roles scoped to the required APIs.
6. Authenticate workloads via Workload Identity (GKE), Workload Identity Federation (external CI / non-GCP compute), or attached service accounts (Cloud Run, Cloud Functions, GCE); do not download service-account JSON keys for workload authentication.
7. Enforce the `iam.disableServiceAccountKeyCreation` organization policy in production folders; create user-managed keys only with a documented exception and a rotation schedule.
8. Disable or remove the default Compute Engine and App Engine service accounts' default-grant `roles/editor` binding; assign explicit, scoped service accounts to each workload instead.
9. Store secrets in Secret Manager and grant access via IAM bindings on the secret; do not pass secrets through environment variables baked into images, Terraform `*.tfvars`, or VM metadata.
10. Encrypt sensitive data at rest with Customer-Managed Encryption Keys (CMEK) in Cloud KMS for any data class flagged by the data-classification policy; do not rely on Google-managed keys for those classes.
11. Enable Cloud Audit Logs Admin Activity, System Event, and Policy Denied logs on every project (on by default — do not disable); enable Data Access logs on services handling regulated data.
12. Route logs and metrics into Cloud Logging and Cloud Monitoring with project-scoped log sinks exporting to a centralized logs project (BigQuery, GCS, or Pub/Sub); do not let production logs live only in a workload project's default `_Default` bucket.
13. Place compute (GCE, GKE nodes, Cloud SQL, Cloud Run direct VPC egress) in a customer-managed VPC; do not use the auto-created default VPC for production workloads.
14. Forbid public IP allocation on Cloud SQL, AlloyDB, GKE node pools, and GCE instances in production via the corresponding org policies (`sql.restrictPublicIp`, `compute.vmExternalIpAccess`); reach private resources through private connectivity (Private Service Connect, VPC peering, IAP tunnels).
15. Pin a primary region (and DR region where required) for every workload via resource location constraints and the `gcp.resourceLocations` org policy; do not let services default to multi-region or arbitrary regions implicitly.
16. Apply consistent labels — `env`, `owner`, `cost_center`, `service` — to every labelable resource; do not ship resources with no `env` or `owner` label.
17. Configure billing budgets and budget alerts for every project, with alerts routed to the owning team; do not run a project without a budget attached.
18. Push container images to Artifact Registry; do not use the deprecated Container Registry (`gcr.io`) for new images.
19. Pin GKE clusters to a release channel (`stable` or `regular`) with maintenance windows; do not pin a single static GKE version that requires manual upgrades.
20. Enable Security Command Center (Standard at minimum, Premium / Enterprise where the data warrants) at the organization scope; triage findings in a ticketed workflow and do not silently dismiss them.
