# ADR-0002: Reference-faithful feel, without the original bugs

**Status:** Proposed — 2026-10-16.

## Context

The value of the game is its controls. The study (v0.2) gives the reference's movement and
gravity tables, verified in the public disassembly, and lists the reference's defects: a flawed
fall-speed cap (5 px/frame overshoot), no stop-at-zero clamp, wall and corner clips, wrong warps,
sprite flicker, PAL timing. retro-maze took the same kind of decision (ADR-0002 there, 455 / 500).

Options: **A** reproduce the tables, remove the defects; **B** bit-exact reproduction, defects
included; **C** our own "modern" tuning (coyote time, jump buffering, float physics).

## MCDA

| Criterion | Weight | A Tables, no bugs | B Bit-exact | C Modern tuning |
|---|---|---|---|---|
| Recognisable feel for players of the reference | 35 | 5 — same speeds, jump, momentum | 5 — identical | 2 — feels like another game |
| Fairness / no glitches for the player | 20 | 5 — clips and overshoot gone | 2 — clips kept | 5 |
| Determinism and testability | 20 | 5 — integer tables, clean rules | 4 — the quirks need special tests | 3 — float tuning harder to replay exactly |
| Simplicity of the rules (KISS) | 15 | 4 — a few clean clamps added | 2 — reproducing quirks is extra code | 3 |
| Consistency with the sibling games' policy | 10 | 5 — same policy as retro-maze | 2 | 2 |
| **Total / 500** | 100 | **485** | 345 | 295 |

Points: A = 175 + 100 + 100 + 60 + 50; B = 175 + 40 + 80 + 30 + 20; C = 70 + 100 + 60 + 45 + 20.

Sensitivity: with *fairness* at 0 the gap is still 80 points (A 385, B 305); only an added
"speedrun-exact" goal could flip it, and none exists.

**Fallback:** B (345), as an optional "authentic" toggle if players ask for the quirks. It is a
drop-in variant of the physics module (one flag), not a design change.

## Decision

The GDD §4 tables reproduce the reference values in our units. The defects listed in GDD §11 are
replaced by clean rules: 4 px/frame clamp, clamp to 0 on release, axis-separated collision with
corner tolerance, no warps, fixed 60 Hz.

## Consequences

- Every removed defect is listed in GDD §11 with its replacement rule; each gets a unit test.
- Feel-defining behaviours are kept on purpose (gravity pair until the next jump, cap drop,
  air momentum).
