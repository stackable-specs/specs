---
id: opentofu
layer: delivery
extends: []
---

# OpenTofu

## Purpose

OpenTofu is the open-source, Linux-Foundation-stewarded fork of Terraform that ships infrastructure changes through a plan/apply workflow against a versioned state file — so a sloppy module is a real-world outage waiting to happen, not a typecheck warning. Unconstrained provider versions let `tofu init` pull a breaking 6.0 plugin overnight, an ungated `tofu apply` against a local state file races with whoever runs it next, secrets committed to `*.tfvars` end up in `git log` forever, an unencrypted S3 state object exposes every database password it captured, and a single shared state file across dev/staging/prod turns one bad apply into a multi-environment incident. This spec pins the toolchain (`required_version`, `required_providers`, lock file), the state backend (remote, locked, encrypted), the module layout and source pinning, secret handling, and the CI workflow (`fmt`/`validate`/`plan -out`/`apply <plan>`), so a reader can trust that two engineers running the same configuration produce the same change set.

## References

- **external** `https://opentofu.org/` — OpenTofu home
- **external** `https://opentofu.org/docs/intro/` — OpenTofu introduction
- **external** `https://opentofu.org/docs/language/state/remote/` — Remote state backends
- **external** `https://opentofu.org/docs/language/state/encryption/` — State encryption
- **external** `https://opentofu.org/docs/language/modules/develop/` — Module development conventions
- **external** `https://opentofu.org/docs/language/expressions/version-constraints/` — Version constraint syntax
- **external** `https://opentofu.org/docs/language/settings/` — `terraform {}` settings block (`required_version`, `required_providers`, `backend`)
- **external** `https://opentofu.org/docs/cli/commands/fmt/` — `tofu fmt`
- **external** `https://opentofu.org/docs/cli/commands/validate/` — `tofu validate`
- **external** `https://github.com/aquasecurity/tfsec` — `tfsec` static analyzer
- **external** `https://www.checkov.io/` — `checkov` policy-as-code scanner

## Rules

1. Pin the OpenTofu version with `required_version` inside a `terraform { ... }` block; do not allow infrastructure to be applied with an unconstrained tool version.
2. Pin every provider in `required_providers` with both an explicit registry `source` and a `version` constraint using a pessimistic operator (e.g. `~> 5.40`) for production root modules; do not declare a provider without an explicit `source` and `version`.
3. Commit the dependency lock file (`.terraform.lock.hcl`) to version control; do not add it to `.gitignore`.
4. Configure a remote backend with state locking (e.g. S3 + DynamoDB, GCS, Azure Blob, or a TACOS such as Spacelift / Scalr / env0); do not run production applies against a local `terraform.tfstate` file.
5. Encrypt state at rest, either via OpenTofu's built-in state encryption or via the backend's server-side encryption (KMS, CMEK, etc.); do not store unencrypted state on cloud object storage.
6. Do not commit `terraform.tfstate`, `.tfstate.backup`, `.terraform/`, `crash.log`, or `*.tfvars` files containing secret values to version control.
7. Lay out each root module with `main.tf`, `variables.tf`, `outputs.tf`, and `versions.tf` (the latter holding `terraform { required_version, required_providers, backend }`); do not place `required_providers` or `backend` blocks in `main.tf`.
8. Pin third-party module sources to a specific version (registry: `version = "X.Y.Z"`; git: `?ref=v1.2.3` or `?ref=<commit-sha>`); do not consume third-party modules from a default-branch HEAD.
9. Mark sensitive input variables and outputs with `sensitive = true`; do not return a secret through an output without the `sensitive` flag.
10. Source secret values from a secrets-manager data source (AWS Secrets Manager, GCP Secret Manager, Vault, etc.) or via `TF_VAR_<name>` environment variables; do not commit secret values to `*.tf`, `*.tfvars`, or `*.auto.tfvars` files.
11. Run `tofu fmt -check -recursive` and `tofu validate` in CI; do not merge configuration that fails either check.
12. Apply production changes by running `tofu plan -out=plan.tfplan` followed by `tofu apply plan.tfplan`; do not run `tofu apply` against live state without an approved plan artifact.
13. Authenticate to cloud providers from CI via OIDC / workload-identity federation; do not store long-lived cloud credentials as CI secrets when an OIDC integration exists.
14. Isolate environments (dev / staging / prod) into separate workspaces, separate state files, or separate root modules; do not share a single state file across environments by switching behavior with conditionals.
15. Apply the team's baseline tag set (e.g. environment, owner, cost-center) to every taggable resource via the provider's `default_tags` block where available; do not ship resources to production without baseline tags.
16. Iterate over named resources with `for_each` keyed on a stable identifier; do not use `count` for a list whose membership may change, since index shifts force unrelated resources to be recreated.
17. Run a configuration security scanner (`tfsec`, `checkov`, or equivalent) in CI against every root module; do not merge changes that introduce new findings above the configured severity threshold.
