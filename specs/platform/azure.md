---
id: azure
layer: platform
extends: []
---

# Microsoft Azure

## Purpose

Azure is a multi-tenant cloud platform whose defaults inherit decades of enterprise tooling — and decades of footguns. A fresh subscription will happily provision a storage account with anonymous blob access, a SQL server reachable from the public internet, a service principal with a non-expiring secret, and a Contributor assignment scoped to the whole subscription, and every one of those defaults has been a real-world breach. Without an opinionated baseline, teams drift: portal-applied changes diverge from IaC, long-lived storage account keys leak through `git push`, dev workloads share a subscription with prod and trip across each other, Log Analytics retention defaults to 30 days while the cost budget defaults to "Never alert," and the Global Administrator account becomes the daily driver. This spec pins the cross-service Azure baseline — multi-subscription management-group structure, IaC-managed provisioning, Entra ID with PIM for humans, Managed Identities and OIDC for workloads, region allowlisting via Azure Policy, customer-managed keys for sensitive data, Private Endpoints over public network access, baseline tagging, log retention, cross-region backups, and per-subscription budget alerts — so individual service specs (Azure SQL, App Service, AKS, Functions, etc.) can refine on top of a known floor instead of redefining it.

## References

- **external** `https://azure.microsoft.com/en-us` — Azure home
- **external** `https://learn.microsoft.com/en-us/azure/well-architected/` — Azure Well-Architected Framework
- **external** `https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/` — Cloud Adoption Framework
- **external** `https://learn.microsoft.com/en-us/security/benchmark/azure/` — Microsoft Cloud Security Benchmark
- **external** `https://learn.microsoft.com/en-us/azure/governance/management-groups/overview` — Management Groups
- **external** `https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/pim-configure` — Entra ID Privileged Identity Management
- **external** `https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/overview` — Managed Identities
- **external** `https://learn.microsoft.com/en-us/azure/governance/policy/overview` — Azure Policy
- **external** `https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-cloud-introduction` — Microsoft Defender for Cloud
- **external** `https://learn.microsoft.com/en-us/azure/key-vault/general/overview` — Azure Key Vault
- **external** `https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-overview` — Private Endpoints
- **external** `https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/cost-mgt-best-practices` — Cost Management best practices

## Rules

1. Manage production Azure resources via Infrastructure as Code (OpenTofu / Terraform with the AzureRM or AzAPI provider, Bicep, or ARM templates); do not provision production resources through the Azure Portal. (refs: opentofu)
2. Use Azure Management Groups with separate subscriptions for production, non-production, and platform/connectivity workloads; do not run production and non-production workloads in the same subscription.
3. Federate human access through Microsoft Entra ID backed by the organization's identity provider, and grant elevated roles only via Privileged Identity Management (PIM) just-in-time activation; do not assign permanent Owner or Contributor roles to individual user accounts.
4. Authenticate workloads via Managed Identities (system- or user-assigned) for in-Azure code paths and federated credentials (OIDC) for CI; do not embed long-lived service principal secrets, storage account keys, or SAS tokens in source, config, container images, or CI secret stores.
5. Pin each workload's deployment regions explicitly and enforce the allowlist via Azure Policy assigned at the management-group scope; do not deploy resources to a region the team has not approved.
6. Enable Azure Activity Log export and resource diagnostic settings to a centralized Log Analytics workspace (or Microsoft Sentinel) in a dedicated platform subscription; do not operate a subscription without Activity Log export.
7. Enable Microsoft Defender for Cloud's standard tier on every subscription with the Microsoft Cloud Security Benchmark assigned; do not silence Defender for Cloud recommendations without a documented, time-bounded exception.
8. Encrypt data at rest with customer-managed keys (CMK) in Azure Key Vault for production data tiers (Storage, Azure SQL, Cosmos DB, managed disks); do not rely solely on platform-managed keys for production data classified as sensitive.
9. Disable public network access by default on storage accounts, Azure SQL servers, Key Vaults, and Cosmos DB accounts — require Private Endpoints or service-tag-restricted firewall rules; do not leave a storage account or database publicly reachable on the internet.
10. Set `allowBlobPublicAccess = false` on every storage account; do not create anonymously readable blob containers without a documented, reviewed exception.
11. Lock Entra ID Global Administrator and subscription-Owner emergency-access ("break-glass") accounts behind separate-from-everyday-use credentials with hardware MFA; do not perform routine operations as Global Administrator.
12. Do not create Network Security Group rules that allow `*` or `Internet` source ingress on application ports; route public traffic through Azure Front Door, Application Gateway, or Azure Load Balancer with a WAF where applicable.
13. Store secrets in Azure Key Vault and retrieve them at runtime via the workload's Managed Identity (or Key Vault references in App Service / Container Apps / Functions); do not place secrets in App Service application settings as plain values, ARM/Bicep parameters, container image layers, or pipeline variables that lack the secret flag.
14. Apply the team's baseline tag set (e.g. `Environment`, `Owner`, `CostCenter`, `Application`) to every taggable resource and to its parent resource group; do not deploy resources to production missing the baseline tag set.
15. Set an explicit retention period on every Log Analytics workspace and a lifecycle-management policy on every storage account that holds logs or data; do not let log retention default to "Never expire."
16. Enable automated backups with cross-region restore for production stateful resources (Azure SQL long-term retention + geo-redundant backup, Cosmos DB continuous backup with multi-region writes or replicas, Azure Backup for VMs and managed disks, GRS/RA-GRS for irreplaceable blob data); do not rely on a single-region copy as the sole backup.
17. Configure an Azure Cost Management budget with at least one alert per subscription wired to an on-call channel; do not run a production subscription without a budget alert.
18. Scope RBAC role assignments to the smallest scope (resource → resource group → subscription → management group) that satisfies the requirement, and prefer built-in least-privilege roles (or custom roles) over Owner/Contributor; do not assign Owner or Contributor at subscription or management-group scope to principals that are not designated break-glass.
