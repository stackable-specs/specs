---
id: beads
layer: practices
extends: []
---

# Beads

## Purpose

Beads (`bd`) is a distributed graph issue tracker that gives AI agents persistent, dependency-aware task memory backed by a version-controlled SQL database (Dolt). Without shared conventions, agents claim the same task concurrently, start work on blocked tasks, leave completed tasks open, and lose context across sessions — producing redundant work and broken execution order. This spec establishes the rules that keep Beads-driven workflows sequential, conflict-free, and auditable.

## Do

- Run `bd ready` to find your next task — only start work on tasks with no open blockers.
- Claim a task atomically with `bd update <id> --claim` before beginning work.
- Model blocking relationships before starting child work: `bd dep add <child> <parent>`.
- Close tasks immediately upon completion with `bd update <id> --status closed`.
- Use server mode when multiple agents write to the same repository concurrently.

## Don't

- Start work on a task that has open blockers.
- Begin work on a task without first claiming it — unclaimed tasks are available to any other agent.
- Leave completed tasks in `in_progress` status.
- Use `relates_to` as a catch-all — choose the specific link type that describes the relationship.

## References

- **external** `https://github.com/gastownhall/beads` — Beads repository, full README and command reference

## Rules

1. Run `bd init` once at the repository root before issuing any other `bd` commands.
2. Use `bd ready` to identify the next task to work on; do not start a task that has open blocking dependencies.
3. Run `bd update <id> --claim` to atomically claim a task before starting work; do not begin work on any task that has not been claimed.
4. Add blocking dependencies with `bd dep add <child> <parent>` before starting the child task; never start a task whose parent blocker is still open.
5. Decompose work that spans multiple sessions or agents into an epic → tasks → sub-tasks hierarchy using Beads' hierarchical ID format (`bd-xxxx`, `bd-xxxx.1`, `bd-xxxx.1.1`).
6. Close completed tasks immediately with `bd update <id> --status closed`; do not leave finished tasks in `in_progress`.
7. Use server mode (external `dolt sql-server`) when two or more agents write to the same Beads database concurrently; use embedded mode only for single-writer workflows.
8. Use `bd init --stealth` on shared projects where Beads metadata should not appear in repository commits.
9. Use `bd init --contributor` on forks to route planning tasks to a separate location rather than the upstream repository.
10. Choose the specific link type when creating relationships: `relates_to` for general associations, `duplicates` for identical issues, `supersedes` when a task replaces another, and `replies_to` for message threading.
