# ADR-0005: Test strategy

**Status:** Proposed — 2026-10-16.

## Context

The feel is defined by numeric tables (GDD §4) and the simulation must be deterministic.
The global testing strategy (test pyramid, FIRST, mock only the boundaries) and the pre-merge
gate (coverage ≥ 80 %, target ~100 %, doc coverage ≥ 90 %) apply.

Options: **A** pyramid — domain unit tests, headless level-simulation tests with scripted inputs
one browser smoke test; **B** mostly browser E2E tests; **C** unit tests only.

## MCDA

| Criterion | Weight | A Pyramid + replay | B E2E-heavy | C Unit only |
|---|---|---|---|---|
| Catches feel regressions (jump arcs, skids) | 30 | 5 — replay compares frame-exact trajectories | 3 — slow, flaky on timing | 3 — no whole-level check |
| Speed and reliability (FIRST) | 25 | 5 | 1 | 5 |
| Mock only the boundaries | 20 | 5 — ports faked, domain real | 4 | 4 |
| Proves the real build runs | 15 | 4 — one smoke test | 5 | 0 |
| Cost to maintain | 10 | 4 | 2 | 5 |
| **Total / 500** | 100 | **475** | 290 | 345 |

Points: A = 150 + 125 + 100 + 60 + 40; B = 90 + 25 + 80 + 75 + 20; C = 90 + 125 + 80 + 0 + 50.

Sensitivity: B overtakes A only if *Proves the real build runs* weighs over 60, more than four
times its current weight.

**Fallback:** C (345), unit-only: a drop-in (delete the smoke test) that leaves the real build
unchecked.

## Decision

- **Unit (Vitest), domain:** every GDD §4 table row, every §11 replacement rule, collision,
  enemies, scoring. Feel values are tested against the study numbers.
- **Simulation (Vitest, headless):** scripted input → recorded hero trajectory and
  score; the same script must replay identically (golden files).
- **Smoke (Playwright):** one test on the real build, every browser engine, title → playing.
- Render and audio are mocked at their ports, never the domain.

## Consequences

- The domain exposes an in-place step, `step(run, inputSnapshot)`, with no hidden clock; the app
  reads `InputPort` and passes the snapshot.
- Golden trajectories are regenerated only by an explicit, reviewed command.
