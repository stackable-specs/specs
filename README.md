# stackable-specs

A layered system model for organizing specifications. Each layer answers exactly one question about a system, so specs compose cleanly and stay independently swappable.

## Philosophy

Most spec repositories tangle concerns — runtime decisions bleed into interface decisions, delivery bleeds into security, and the result is a pile of documents no one can navigate. `stackable-specs` enforces a single rule: **one layer, one question.** If a spec doesn't answer the layer's question, it belongs in a different layer.

The layers are ordered from *what a system is made of* (language) up through *how we work on it* (practices, quality). Higher layers may reference lower layers; lower layers never assume higher ones.

## Layers

| Layer | Question it answers |
| --- | --- |
| [`language`](specs/language) | What is it written in? |
| [`platform`](specs/platform) | Where does it run? |
| [`interface`](specs/interface) | How is it invoked? |
| [`presentation`](specs/presentation) | What does it output? |
| [`delivery`](specs/delivery) | How is it shipped? |
| [`observability`](specs/observability) | How is it inspected? |
| [`security`](specs/security) | What must never break trust? |
| [`practices`](specs/practices) | How do we work? |
| [`quality`](specs/quality) | How do we enforce? |

## Repository layout

```
/specs
  /language         # syntax, idioms, style guides, language-version pins
  /platform         # OS, runtime, hardware, cloud, container targets
  /interface        # APIs, CLIs, protocols, event contracts, SDK surfaces
  /presentation     # UI, formatting, rendering, output shapes
  /delivery         # build, package, release, deploy, rollback
  /observability    # logs, metrics, traces, alerting, dashboards
  /security         # authn, authz, secrets, data handling, threat model
  /practices        # workflow, review, branching, decision-making
  /quality          # testing, linting, gating, SLOs
```

## How to use it

- **Adding a spec:** decide which question it answers, then drop it in that layer. If it answers two, it's really two specs — split them.
- **Reading a spec:** you can read any single layer in isolation and get a coherent view. To understand a full system, read bottom-up.
- **Swapping a layer:** because layers don't leak into each other, replacing the `platform` specs (say, moving from VMs to containers) should not require edits in `interface` or `presentation`.

## Strengths

- **Clean mental model.** Every spec has an obvious home.
- **Maps to real systems.** The layers line up with how teams already divide ownership.
- **Highly composable.** Specs can be mixed and matched across projects without rewriting.

## Status

Early scaffolding. Each layer directory is currently empty; specs will be added as the model is exercised against real systems.
