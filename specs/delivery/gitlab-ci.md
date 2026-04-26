---
id: gitlab-ci
layer: delivery
extends: []
---

# GitLab CI/CD

## Purpose

GitLab CI/CD is the pipeline substrate that decides what merges to the default branch, what container images publish, and what credentials a job may exercise — so a sloppy `.gitlab-ci.yml` is simultaneously a supply-chain risk, a secret-leak risk, and a wasted-runner-minute liability. Floating `image: foo:latest` references re-pull arbitrary content on every run, unpinned `include: component:` pulls inherit whatever upstream pushes today, deprecated `only:` / `except:` clauses produce subtly different filtering than a `rules:` reader expects, masked but unprotected variables leak into branch pipelines, and an unscoped `$CI_COMMIT_MESSAGE` interpolation hands a contributor a shell on the runner. This spec pins how pipelines are structured (stages, includes, components), how jobs are scoped (rules, environments, runners, resource groups), how secrets and untrusted input are handled, and which keywords are deprecated, so "the pipeline passed" is a meaningful gate rather than a coincidence.

## References

- **external** `https://docs.gitlab.com/ci/` — GitLab CI/CD documentation
- **external** `https://docs.gitlab.com/ci/yaml/` — `.gitlab-ci.yml` keyword reference
- **external** `https://docs.gitlab.com/ci/yaml/deprecated_keywords/` — Deprecated keywords
- **external** `https://docs.gitlab.com/ci/components/` — CI/CD Components and the catalog
- **external** `https://docs.gitlab.com/ci/runners/` — Runner types, tags, and executors
- **external** `https://docs.gitlab.com/ci/variables/` — CI/CD variables, masking, and protection
- **external** `https://docs.gitlab.com/ci/secrets/` — External secrets and ID tokens (OIDC)
- **external** `https://docs.gitlab.com/ci/environments/` — Deployment environments and protection rules
- **external** `https://docs.gitlab.com/ci/yaml/script/#use-special-characters-with-script` — Script injection guidance

## Rules

1. Define every pipeline as `.gitlab-ci.yml` at the repository root, with included fragments under a versioned directory (e.g. `.gitlab/`); do not store CI logic outside committed YAML.
2. Declare a top-level `stages:` list to enforce explicit stage ordering; do not rely on implicit stage assignment.
3. Use `rules:` to control when jobs run; do not use the deprecated `only:` / `except:` keywords in new pipelines.
4. Pin third-party CI/CD components (`include: component:`) to a commit SHA or a released semantic version tag (e.g. `@v1.4.2`); do not reference third-party components by branch name or `@~latest`.
5. Pin third-party `include: remote:` URLs to a tagged or commit-pinned ref; do not include remote YAML from an unpinned default branch.
6. Pin container images referenced by `image:` and `services:` to a specific tag (preferably with a digest, e.g. `ruby:3.3.4@sha256:<digest>`); do not use the `:latest` tag for build, test, or deploy jobs.
7. Set an explicit `timeout:` on every job that performs network, build, or deploy work; do not let long-running jobs rely solely on the project's default 60-minute timeout.
8. Set `interruptible: true` on test, lint, and other re-runnable jobs so newer pipelines supersede older ones; do not mark deploy or release jobs interruptible.
9. Coordinate mutually-exclusive deploy jobs with `resource_group:`; do not allow two pipelines to deploy to the same environment concurrently.
10. Store secrets as masked, protected CI/CD variables (or as `file:`-type variables for tools that read credentials from disk); do not commit credentials to `.gitlab-ci.yml` and do not store production secrets in a non-protected variable.
11. Authenticate to external secret managers and cloud providers via the `secrets:` keyword and GitLab ID tokens (OIDC) for production credentials; do not store long-lived cloud credentials as masked CI/CD variables when an OIDC integration exists.
12. Assign user-controlled values (`$CI_COMMIT_MESSAGE`, `$CI_MERGE_REQUEST_TITLE`, `$CI_COMMIT_BRANCH`, contributor-supplied variables) to an intermediate variable and reference that variable from `script:`; do not interpolate user-controlled values directly inside shell commands.
13. Declare an `environment:` (with `name:` and `url:`) on every deploy job; do not deploy to production from a job that has no `environment:` declaration.
14. Gate production deploys with `when: manual` against a protected environment that requires approval; do not auto-deploy to production from a non-protected branch.
15. Bind sensitive jobs to project- or group-scoped runners via `tags:`; do not run production deploy jobs on shared runners that also serve untrusted forks.
16. Cache language and toolchain dependencies with `cache:` whose `key:` is derived from the project's lockfile; do not configure a cache without a key tied to dependency state.
17. Declare `artifacts:` with an explicit `expire_in:` for every job that publishes artifacts; do not rely on the project default expiration for ephemeral build outputs.
