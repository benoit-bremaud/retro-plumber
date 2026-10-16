# Project Log — retro-plumber

Reverse-chronological operational logbook (most recent first). It complements `git log` with the
human context: what was done, why, and by which PR.

---

## 2026-10-16

### Brick 0 — repository skeleton (branch `docs/study`)

- Owner decision: a side-scrolling platformer in the tradition of the first 1985 home-console
  platformer, codename retro-plumber (owner choice over MCDA's retro-platformer 440 / 500).
  Scope: one overworld level as vertical slice first (MCDA 385). Study only now; implementation
  waits for `../retro-shmup`'s vertical slice.
- Directory layout mirrors retro-shmup (MCDA 475 vs full four-ring Clean 350, flat 320):
  `src/{app,adapters,domain/ports,...}`, `tests/`, `docs/{decisions,design,study,architecture}`.
- Owner rule: Clean architecture everywhere and SOLID always — written into CLAUDE.md.
- Skeleton: README, MIT LICENSE, CLAUDE.md, `.gitignore`, this log, `.gitkeep` placeholders.
- Reference study v0.1 (`docs/study/reference-study.md`): fixed-point units, walk/run caps,
  friction table, speed-dependent jump and gravity, variable jump height, generic cast, level
  grammar, bugs to drop, 5 open questions. Sourced values tagged [src]/[int]/[gen].
- Reference study v0.2: physics checked against doppelganger's public disassembly (read only,
  nothing copied). Units, gravity and jump tables confirmed; acceleration-index mapping per
  situation established; the cap is a hard clamp; terminal fall is 4 px/frame with a one-frame
  5 px/frame overshoot. Still open: the stop-at-zero rule.
- Skid end located (`ProcSkid`: opposite input and |speed| < 11 → stop and flip); no explicit
  stop-at-zero on release — our design clamps to 0.
- GDD v0.1: vision, IP boundary, cast *(proposed: Wrench, Mudlet, Clank, Snapvine)*, movement
  tables in our units, collision, enemies, level format, scoring, flow, vertical slice, removed
  bugs.
- ADR-0001 stack (460 vs Phaser 340), ADR-0002 feel without bugs (485), ADR-0003 module reuse
  deferred to the first code brick — rule of three now met (430 vs copy 260), ADR-0004 codename
  retro-plumber (owner choice, fallback retro-platformer 440), ADR-0005 tests (475), ADR-0006
  layout (475; flat option corrected 310 → 320). Each ADR records its runner-up as fallback.
- UML (Mermaid, render-checked with mmdc): use cases, game flow, components, domain classes,
  hero states.
- Pending: pre-push review; pre-push review; owner approval to create the public GitHub
  repository and the PR.
