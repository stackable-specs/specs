---
id: gitflow
layer: practices
extends: []
---

# GitFlow

## Purpose

GitFlow is Vincent Driessen's branching model for projects that ship as discrete, versioned releases — installed software, mobile apps held by store review, embedded firmware, SDKs with multi-version support windows — where a stabilization phase, a tagged production line, and the ability to back-port a fix to a still-supported earlier version are real requirements rather than ceremony. The model only delivers those properties when its lifecycle is followed strictly: `main` carries only tagged releases, `develop` is the integration branch, every `release/*` merge fans out to *both* `main` and `develop`, every `hotfix/*` does the same, and short-lived feature branches do not bleed into `main`. Where teams skip the dual-merge, fix-forward on a release branch without back-porting, squash-merge into `main` and lose the topology, branch hotfixes from `develop` instead of from a production tag, or let a "release" branch live for months absorbing new features, GitFlow degrades into a network of long-running branches whose merge cost outpaces any release-stabilization benefit. This spec pins the branch taxonomy, the strict lifecycle of each branch type, the dual-merge requirement for `release/*` and `hotfix/*`, the SemVer tagging discipline on `main`, and the explicit applicability boundary against continuously-deployed SaaS — so a team that chose GitFlow because it actually fits them runs it correctly, and a team for whom it does not fit is steered to `trunk-based-development` instead.

## References

- **spec** `trunk-based-development` — sibling practices-layer branching model for continuously-deployed services
- **spec** `conventional-commits` — practices-layer commit format that pairs with GitFlow's release tagging
- **external** `https://nvie.com/posts/a-successful-git-branching-model/` — Vincent Driessen: "A successful Git branching model" (the original GitFlow article)
- **external** `https://github.com/petervanderdoes/gitflow-avh` — `git-flow` AVH CLI extension
- **external** `https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow` — Atlassian: GitFlow workflow
- **external** `https://semver.org/` — Semantic Versioning 2.0.0
- **external** `https://martinfowler.com/articles/branching-patterns.html` — Patterns for managing source code branches (Fowler)

## Rules

1. Apply this spec only to projects that ship as discrete versioned releases (installed software, mobile apps, embedded firmware, SDKs with support windows); for continuously deployed services, adopt `trunk-based-development` instead. (refs: trunk-based-development)
2. Maintain two long-lived branches — `main` (alias `master`, production releases only) and `develop` (integration); do not introduce a third long-lived branch such as `staging`, `qa`, or a perpetual `release`.
3. Every commit on `main` represents a tagged production release; do not push commits to `main` that are not the result of a `release/*` or `hotfix/*` merge.
4. Tag every merge to `main` with an annotated SemVer tag (`vX.Y.Z`) in the same operation as the merge; do not merge to `main` without simultaneously creating the version tag.
5. Name feature, release, and hotfix branches with the documented prefix (`feature/`, `release/`, `hotfix/`) and a meaningful slug; do not commit to a topic branch that lacks the prefix.
6. Branch `feature/*` from `develop` and merge it back to `develop` only; do not merge a `feature/*` branch into `main`, `release/*`, or `hotfix/*`.
7. Branch `release/<version>` from `develop` when stabilization begins and accept only bug fixes, documentation, version bumps, and release-prep commits on it; do not land new feature work on a `release/*` branch.
8. Merge every `release/*` branch into both `main` (producing the tagged release) and back into `develop` (so stabilization fixes flow forward); delete the release branch immediately after the dual-merge.
9. Branch `hotfix/<version>` from the affected production tag on `main`; do not branch a hotfix from `develop` or from another long-lived branch.
10. Merge every `hotfix/*` branch into both `main` (with a new `PATCH` tag) and back into `develop` — or into the active `release/*` branch if one exists, which then carries the fix into `develop` at its own merge; delete the hotfix branch immediately after the dual-merge.
11. Use merge commits (`git merge --no-ff`) when merging `feature/*` → `develop`, `release/*` → `main`, `release/*` → `develop`, `hotfix/*` → `main`, and `hotfix/*` → `develop`; do not squash-merge or rebase-merge into the long-lived branches (the merge-commit topology is part of GitFlow's audit trail).
12. Increment the SemVer `MAJOR` for breaking changes, `MINOR` for new features released from a `release/*` branch, and `PATCH` for `hotfix/*` releases; do not assign version numbers ad-hoc. (refs: conventional-commits)
13. Protect `main` and `develop` with branch-protection rules requiring passing CI and at least one approving review; do not allow direct pushes to either long-lived branch.
14. Forbid force-pushes and history rewrites to `main` and `develop`; do not rebase a long-lived branch.
15. Delete every `feature/*`, `release/*`, and `hotfix/*` branch immediately after the merge that completes its lifecycle; do not let topic branches accumulate after they have served their purpose.
16. Run the full CI suite on every push to `feature/*`, `release/*`, and `hotfix/*` branches; block merges into `main` or `develop` when CI is red.
17. Document active `release/*` and `hotfix/*` branches (their version, owner, and target ship date) in a known location (CHANGELOG, release-management page, repo README); do not let in-flight stabilization branches exist that the rest of the team cannot find.
18. Drive the `release/*` and `hotfix/*` lifecycles through the `git-flow` CLI extension (or equivalent scripted automation) so the dual-merge and tag step are not skipped; do not rely on humans to remember the second merge.
