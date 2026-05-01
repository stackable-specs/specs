---
id: py-import-order
layer: language
---

# Python: Import Order

## Purpose

Consistent import order keeps diffs small, makes merges predictable, and reduces cognitive load when scanning a file. Without a shared rule, contributors reach for different defaults (alphabetical, dependency order, by category) and the codebase churns.

## References

- **external** `https://peps.python.org/pep-0008/#imports` — PEP 8 import guidance
- **external** `https://pycqa.github.io/isort/` — isort configuration reference
- **adr** `ADR-014` — decision to adopt the black profile for isort

## Rules

1. Group imports in three blocks separated by a single blank line: standard library, third-party, first-party. (refs: https://peps.python.org/pep-0008/#imports)
2. Sort imports alphabetically within each group.
3. Place `import x` statements before `from x import y` statements within the same group.
4. Do not use wildcard imports (`from x import *`) outside of `__init__.py` re-exports.
5. Run `isort` with the `black` profile as part of pre-commit. (refs: ADR-014)
