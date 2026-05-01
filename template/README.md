# Stack Template

Bare-bones scaffold for a new stack. Every directory exists because of an ADR.
Add language, framework, and toolchain specs under `docs/specs/` as you make decisions.

## Repository layout

```
.
├── .github/
│   └── workflows/          # CI/CD pipelines
├── docs/
│   ├── adr/                # Architectural Decision Records (MADR format)
│   │   ├── README.md
│   │   └── 000-template.md
│   ├── bdr/                # Behavior Decision Records
│   │   ├── README.md
│   │   └── 000-template.md
│   └── specs/              # Specs that ADRs reference (copied from stackable-specs)
│       ├── delivery/
│       │   ├── docker.md
│       │   └── github-actions.md
│       ├── practices/
│       │   ├── conventional-commits.md
│       │   ├── git.md
│       │   ├── madr.md
│       │   └── tdd.md
│       ├── quality/
│       │   └── unit-testing.md
│       └── security/
│           └── dependency-management.md
├── src/                    # Application source code
├── tests/                  # Automated tests
├── verify/                 # Smoke / post-deploy verification scripts
├── .gitignore
├── Makefile
└── README.md
```

## Included specs

| Spec | Layer | Why it's essential |
| ---- | ----- | ------------------ |
| `practices/madr.md` | practices | ADR format and lifecycle |
| `practices/bdr.md` | practices | Behavior record format and lifecycle |
| `practices/conventional-commits.md` | practices | Commit message contract |
| `practices/tdd.md` | practices | Red-green-refactor discipline |
| `practices/git.md` | practices | Branch and merge workflow |
| `quality/unit-testing.md` | quality | Unit-test scope and naming rules |
| `security/dependency-management.md` | security | Dependency policy |
| `delivery/docker.md` | delivery | Container image conventions |
| `delivery/github-actions.md` | delivery | CI/CD pipeline conventions |

## Getting started

1. Copy this template into your new repository.
2. Write ADR-001 selecting your language (use `docs/adr/000-template.md`).
3. Copy additional specs from `stackable-specs/specs/` as each ADR references them.
4. Delete `.gitkeep` files once the directories have real content.
