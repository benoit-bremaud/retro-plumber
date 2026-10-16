# ADR-0003: Reuse of retro-shmup modules — decision deferred to the first code brick

**Status:** Proposed — 2026-10-16.

## Context

retro-shmup contains generic, tested modules (frame loop, clock, seeded RNG, ports, input with
rebinding, save, hall of fame, name entry, i18n, audio synth). retro-maze (ADR-0003 there) and
retro-blocks (ADR-0002 there) decided to **copy and adapt** them, and retro-maze wrote that the
choice must be revisited when a third game needs them (rule of three). retro-plumber is at least
the fourth consumer: the rule of three is now met.

But retro-plumber writes no code before retro-shmup's vertical slice (owner decision 2026-10-16).
The modules will still change in retro-shmup until then, and the other games' copies will have
shown how much they drift.

Options: **A** copy and adapt now (record a source SHA); **B** decide now to extract a shared
package; **D** defer the decision to the first code brick of retro-plumber, with this matrix as
input.

## MCDA

| Criterion | Weight | A Copy now | B Shared package now | D Defer |
|---|---|---|---|---|
| Decision on real evidence (drift seen in the copies) | 30 | 2 — decided before the evidence exists | 2 — same | 5 — decided when the drift is known |
| DRY across 4+ games | 25 | 1 — a fourth copy | 5 | 3 — leaves the door open |
| No work wasted before code exists (YAGNI) | 25 | 3 — SHA recorded, soon stale | 1 — package work now for no code | 5 |
| Decoupling between games | 20 | 5 | 2 | 4 — chosen later on evidence |
| **Total / 500** | 100 | 260 | 250 | **430** |

Points: A = 60 + 25 + 75 + 100; B = 60 + 125 + 25 + 40; D = 150 + 75 + 125 + 80.

Sensitivity: B overtakes D only if *DRY* becomes the dominant criterion, which would also make it the
decision for the sibling games — a cross-project choice, not one for this repository alone.

**Fallback:** A (260), copy and adapt as the siblings did: a drop-in (copy the files, record
the SHA, current retro-shmup `main` is `4d75caf`).

## Decision

No module is copied in brick 0. The first code brick of retro-plumber reopens this ADR with the
observed drift between the copies in retro-maze and retro-blocks, and picks copy or shared
package by MCDA.

## Consequences

- Nothing to maintain until code starts.
- The shared-package question becomes cross-project: a backlog issue should be filed on
  retro-shmup when this repository exists on GitHub.
