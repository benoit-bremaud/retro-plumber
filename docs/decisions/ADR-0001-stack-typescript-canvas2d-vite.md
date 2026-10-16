# ADR-0001: Stack — TypeScript strict, native Canvas 2D, Vite

**Status:** Proposed — 2026-10-16.

## Context

retro-plumber is a 256 × 240 tile-based side-scroller with one hero, a few dozen enemies per level
and a scrolling camera. Its hard requirements (CLAUDE.md, GDD §12) are a fixed 60 Hz
deterministic simulation in fixed-point integers, a domain that never imports the browser, no
allocation in the loop, and headless simulation tests. Three sibling projects (retro-shmup,
retro-maze, retro-blocks) already chose TypeScript + Canvas 2D + Vite against the same
requirements (450–475 / 500 there).

Options: **A** TypeScript strict + native Canvas 2D + Vite; **B** Phaser 3 (TS, Arcade Physics);
**C** Godot 4 with web export.

## MCDA

| Criterion | Weight | A Canvas 2D | B Phaser | C Godot |
|---|---|---|---|---|
| Determinism / exact physics control | 30 | 5 — we own the loop and integer physics | 3 — Arcade Physics is float-based; we would bypass it | 3 — engine physics float-based, custom body needed |
| Consistency with the sibling games | 20 | 5 — same toolchain ×3 | 2 — new framework | 0 — other language |
| Headless testability (Vitest) | 20 | 5 — pure domain | 4 — domain testable, scene glue not | 1 — engine runtime required |
| Bundle size / first load | 10 | 5 — tens of kB | 3 — ~1 MB | 0 — tens of MB of wasm |
| Ecosystem for free (tilemaps, camera, editor) | 20 | 3 — we write the tilemap and camera (small) | 5 — tilemaps and camera built in | 5 — full editor |
| **Total / 500** | 100 | **460** | 340 | 210 |

Points: A = 150 + 100 + 100 + 50 + 60; B = 90 + 40 + 80 + 30 + 100; C = 90 + 0 + 20 + 0 + 100.

Sensitivity: even with *Ecosystem* at 40 (and *Determinism* at 10), A = 420 vs B = 380 — A stays
ahead. The platformer case is more favourable to A than the siblings: the reference feel depends
on exact integer tables that a float physics engine would fight.

**Fallback:** B, Phaser 3 (340): the next best if our own tilemap or camera proved too costly.
Activation cost: a design change (render, input and camera adapters rewritten around Phaser
scenes); the pure domain is untouched by construction.

## Decision

TypeScript in strict mode, native Canvas 2D behind a `RenderPort`, Vite for dev server and build,
Vitest for tests, Playwright for the one smoke test, pnpm. No game framework or rendering library.

## Consequences

- Same toolchain as the sibling games: one mental model, copyable configs.
- We write the tilemap renderer and the camera ourselves (small, well-understood code).
- Adding a framework later requires a superseding ADR (CLAUDE.md *Forbidden*).
