# Project Log — retro-plumber

Reverse-chronological operational logbook (most recent first). It complements `git log` with the
human context: what was done, why, and by which PR.

---

## 2026-10-22 (brick 5, branch `docs/uml-sequences`, issue #7)

- Sequence diagrams for the critical flows: 10 collisions (stomp, block hit), 11 item pickup, 12 flag and tally, 13 death and retry, 14 pause and resume. Traceability matrix v0.2 (UC1/UC2 cells, reverse check, hand-off of 13 to UC3/UC4).
- MCDA D3 five diagrams (B 415 vs C 390, fragile: C leads when coverage weighs 15 and KISS 35, recorded fallback C) and D4 module-level participants (A 450 vs B 275, fallback B).
- review-local green (docs-only): Must 0, Should 10 fixed, report `docs/architecture/review-local.md`. Codex ran 14 review rounds on the PR (about 35 inline findings, all valid P2 on call/return semantics, scene transitions and missing branches); each answered inline, threads left open. Resulting conventions: every `step` returns once to `FrameLoop`, terminal transitions happen inside the step, two-bit pause snapshot (toggle, auto-pause), `GameOver` held until confirm or delay (open [gen] point).
- Merged: PR #18 squash-merged (5662451), branch deleted, issue #7 closed. Matrix v0.2 still marked Proposed (no explicit owner acceptance). No CI, SonarQube or branch protection yet.
- Open: OQ1–OQ3 (GDD revision), gaps G1/G3, GameOver duration/confirm rule, class diagram 04 missing `Contacts` dependencies on `Session`/`Hero`/pools, diagram 02 lists Esc as a pause trigger (OQ2). Next: brick 6 (issue #8).

## 2026-10-19 (brick 4, branch `docs/uml-specs`, issue #6)

- Cockburn specifications for UC1–UC6 (`docs/architecture/use-case-specifications.md`) and the
  traceability matrix (`traceability-matrix.md`) with recorded gaps G1–G3 and open questions OQ1–OQ3.
- MCDA D1 single spec file (395 vs 375, fragile under traceability weight 40) and D2 three
  sequences + one data-flow (400 vs 340); weights validated by the owner.
- New diagrams: 06 tick sequence, 07 name entry, 08 rebind, 09 persistence data-flow.
  Brick-3 follow-ups fixed: growth item during invulnerability (hero), pause/Esc/auto-pause triggers (flow).
- review-local green after fixes (Must 2 + Should 12 fixed), PR #16. Owner validated the specs and matrix: status Accepted.
- Merged: PR #16 squash-merged (bc12106), branch deleted. Owner accepted the use-case specifications and the traceability matrix. Local review: Must 2 and Should 12 found and fixed (domain must not call ports; app performs I/O). Codex: 2 inline findings answered. No CI, SonarQube or branch protection yet. Open: OQ1–OQ3 (GDD revision), gaps G1/G3. Next: brick 5, issue #7 (sequence diagrams).

## 2026-10-19

### Brick 3 — GDD v1.0 (issue #5, PR #14, 83428ee)

- GDD v0.1 → v1.0: no *(proposed)* GDD item left; every figure tagged [src]/[int]/[gen]; change log; 14 MCDA matrices in §13 resolving Q1–Q14 (Q4 withdrawn, Q12 never existed, Q14 closed by complement D2).
- Cast renamed for IP safety (name clash search): Mudlet → Grubbit, Clank → Dentshell, Snapvine → Gulpvine (440 vs 360); rename also applied to the level study ids' text.
- Key values: hit invulnerability 158 frames (460), shell revival 280 frames (410), slice timer 400 in the level header (395, narrow), contact test every frame (445), auto-pause (430), flag bands by tile rows (440), hitboxes 12×14 / 12×28 (430).
- Codex review: pause lock keeps counting while paused, touch pause button + resume overlay, `EndSequence` state before `Tally` in the flow diagram; answered inline, threads left open (owner rule).
- Open: scoring values (coin, brick, growth item, time bonus) tagged [gen], to verify against the disassembly at the first code brick. No CI, SonarQube or branch protection yet.

### Brick 2 — level study and slice plan (issue #4, branch `docs/level-study`)

- Structure chosen by the owner: one annex `docs/study/level-study.md` (MCDA 445, fallback two files 395).
- Original 200 × 13 tile-grid layout in six sections (intensity 1, 2, 3, 3, 5, 1) matching the GDD §10 quotas; enemy placement rules E1–E7 written as checkable predicates; jump limits derived from the GDD tables including the 4 px/frame terminal speed (walk ≈ 85 px, run ≈ 139 px).
- MCDA D2 length 450, D3 sawtooth curve 465, D4 rule-based placement 450, D5 one hidden block + optional route 430, each with a fallback.
- Open for the GDD revision: Q8 flag bands mapping, Q9 brick-row and staircase interpretation, Q10 hidden block count, Q11 level time, Q13 pit 3 run-intended vs enforced, Q14 camera trigger wording. Status: Accepted by the owner, PR #13 squash-merged; Codex review: two P2 comments fixed.


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
