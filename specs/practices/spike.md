---
id: spike
layer: practices
extends: []
---

# Spike (Time-Boxed Investigation)

## Purpose

A spike is a time-boxed investigation that exists to retire a specific unknown — "will this library handle our throughput?", "what does the real API response look like?", "is this refactor reachable in one sprint?" — by writing the smallest piece of code or running the smallest experiment that returns a confident yes/no/uncertain. The technique is only valuable when the box is held: when the goal is a written answer, when the code is treated as disposable, and when the spike produces a decision (proceed, abandon, follow-up spike) rather than a half-built feature smuggled into trunk. Without that discipline "spike" becomes the label teams attach to any work too speculative to estimate, time slips silently into days, the prototype gets merged because "it already works," and the question that motivated the spike is never explicitly answered. This spec pins the spike charter (question, time-box, success criteria), the disposable-code rule, the written outcome, and the relationship to ADRs/BDRs and the planning process so spikes stay a tool for retiring uncertainty rather than a category of unaccountable work.

## References

- **spec** `madr` — record format used to capture an architectural decision a spike produced
- **spec** `bdr` — behavior contract a spike's findings may shape
- **spec** `tdd` — once a spike retires the unknown, implementation returns to red-green-refactor
- **external** `https://en.wikipedia.org/wiki/Spike_(software_development)` — Spike (software development)
- **external** `http://www.extremeprogramming.org/rules/spike.html` — Original XP rule on spike solutions
- **external** `https://www.scaledagileframework.com/spikes/` — SAFe guidance on spikes (functional vs technical)
- **external** `https://martinfowler.com/bliki/SpikeSolution.html` — Martin Fowler on spike solutions

## Rules

1. Open every spike with a written charter that states the question to answer (one sentence), the time-box, the success criteria (what evidence ends the spike), and the named owner; do not start a spike whose goal is "explore X" with no question and no exit condition.
2. Make the time-box explicit and bounded — a fixed number of person-hours or person-days, never an open-ended "until we know"; do not start a spike without a number on the budget.
3. Halt the spike when the time-box expires regardless of whether the question is answered; do not silently extend the budget — open a follow-up spike with a new charter if more investigation is needed.
4. Track every spike as its own ticket / issue with the `spike` label (or equivalent) so spikes are visible in planning and reporting alongside feature work; do not hide spikes inside a feature ticket's description.
5. Treat all code written during a spike as disposable; do not allow spike code to be merged to the default branch.
6. Hold spike code on a clearly-named branch (e.g. `spike/<short-question>`) and delete the branch on completion; do not retain spike branches indefinitely as a "reference."
7. Skip the production discipline that does not earn its keep on a throwaway: skip exhaustive tests, skip lint perfection, skip architectural cleanup; do not also skip the discipline that does earn its keep — secret hygiene, dependency provenance, and not running the spike against shared production data.
8. End every spike with a written outcome document (a comment on the spike ticket, a markdown note, or an ADR draft) that records the answer, the evidence (commands run, measurements taken, screenshots, reproducer commit SHA), the resulting decision (proceed / abandon / follow-up spike), and any new unknowns discovered.
9. Capture an architectural decision the spike produced as an ADR per `specs/practices/madr.md`; do not let a spike's "we will use X" outcome live only in a ticket comment.
10. Capture a behavioral contract the spike clarified as a BDR per `specs/practices/bdr.md`; do not let a spike's "the system must do Y" outcome live only in a ticket comment.
11. Re-implement any code that survives the spike from scratch in a separate, properly engineered pull request that follows the project's TDD, lint, and review rules; do not promote the spike branch by adding tests after the fact.
12. Run the spike against an isolated environment (a scratch database, a sandbox account, a containerized stub) when the question requires real I/O; do not run a spike against shared production data, shared production credentials, or third-party billing endpoints without an approved exception.
13. Record an "answer: inconclusive" outcome when the time-box expires without a confident yes/no; do not stretch the language of the outcome to imply more certainty than the evidence supports.
14. Do not use a spike as cover for work that should be a normal feature, a refactor, or a bug fix; if the work is committed to in advance ("we are going to ship X"), it is not a spike — open the appropriate ticket type.
