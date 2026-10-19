# Project Log — retro-plumber

Reverse-chronological operational logbook (most recent first). It complements `git log` with the
human context: what was done, why, and by which PR.

---

## 2026-10-19

### Brick 1 — complementary study (issue #3, branch `docs/study-complement`)

- Structure chosen by the owner: one annex `docs/study/reference-study-complement.md` (MCDA 440, fallback six annexes 420).
- Covers timing, camera, rendering, audio, controls, enemy/block/item/end-of-level detail. Values checked in the public disassembly (read only): 24 frames per level-timer unit, 21-frame interval timers, scroll thresholds 80 / 112 px, walker 0.5 px/frame, kicked shell 3 px/frame, hit invulnerability 148-168 frames, star 714-735 frames.
- Five MCDA choices recorded with fallbacks (D1 frame timers 465, D2 reference camera 480, D3 own palette 455, D4 synth audio 460, D5 fixed touch pad 420).
- Seven open questions for a GDD revision (invulnerability 2.8 s vs 2 s, shell revival, shell revival, …). Status: Accepted by the owner, PR #12 squash-merged.

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
- Local review (design + docs) reconciled: GDD units table and hero fields added; ADR sensitivity
  checks and option labels fixed (ADR-0003 D → C); RNG wording made conditional (no random rule
  in the slice, `src/domain/random` removed); class, hero-state and component diagrams rewritten
  (Session, Hero units, Clock owned by app, Dead/FlagSlide/AutoWalk at hero level). Report:
  `docs/architecture/review-local.md`. Tests, coverage and SonarQube: not applicable (docs-only).
- Merged: public repo `benoit-bremaud/retro-plumber` created, PR #1 squash-merged (23e7cbd), branch deleted.
- Pending: owner acceptance of ADR-0001/0002/0003/0005; SECURITY.md, CONTRIBUTING.md and security CI; cross-project shared-package issue on retro-shmup. Next code brick waits for retro-shmup's vertical slice.
