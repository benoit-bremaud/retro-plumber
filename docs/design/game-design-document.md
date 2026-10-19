# retro-plumber — Game Design Document

**Version:** 1.0 — 2026-10-19 (brick 3, issue #5). **Status:** v1.0 for owner validation; no GDD item is
left *(proposed)* (ADR acceptance is tracked separately in `docs/decisions/`).

Tag legend, on every figure: **[src]** read in a public source (cited by the studies), **[int]** our
derivation from a sourced value, **[gen]** genre knowledge or our own design choice, not checked
against a source. The MCDA matrices behind every v1.0 choice are in §13.

This document is the design contract: no rule is implemented before it is written here, and the
code follows these tables. The behavioural evidence behind the numbers is in
[`docs/study/`](../study/) (main study v0.2, complement v0.3, level study v0.2); technical choices
live in the ADRs under [`docs/decisions/`](../decisions/); the UML study under
[`docs/architecture/`](../architecture/).

## 1. Vision

A side-scrolling platformer for the browser. The player runs right through a level, jumps over
gaps, stomps enemies, breaks bricks, bumps item blocks and reaches the flag before the timer
runs out. The pleasure comes from the controls: momentum, skids and a jump whose height the
player controls with the button.

Pillars:

1. **Reference-faithful feel.** Speeds, accelerations, jump and gravity tables reproduce the
   1985 home-console reference (study §2–§3), so that the controls feel the way players remember.
2. **Without the bugs.** Known defects and quirks are removed (§11, ADR-0002).
3. **Original identity.** Our own character, enemies, levels, art, sound and name (ADR-0004).
4. **Deterministic and testable.** Fixed 60 Hz simulation in fixed-point integers (any future
   random rule draws from a seeded RNG): the same inputs replay the same run (CLAUDE.md, ADR-0005).
5. **Clean architecture.** The rules below live in the pure domain; the browser is an adapter
   (CLAUDE.md § Architecture).

## 2. Behavioural reference and IP boundary

The behavioural evidence is the community documentation and doppelganger's public disassembly,
read for verification only (study § Sources). Nothing from them is copied as code or data: tables
are re-expressed in our own units and structures.

- **Reproduced:** movement rules, speed and gravity values, timings, scoring rules, the general
  grammar of blocks and enemies.
- **Never reproduced:** title, logo, character and enemy names, sprites, tiles, sounds, music,
  level layouts, the castle / princess story. Cast (§3) and levels (§7) are original.

## 3. Cast — names and look are ours

| Role | Name | Behaviour |
|---|---|---|
| Hero — a plumber-engineer | **Pip-Wrench** ("Wrench") | §4 |
| Walker | **Grubbit** | walks, turns at walls, falls off ledges |
| Shelled walker | **Dentshell** | walks; stomp → shell; kicked shell slides |
| Pipe plant | **Gulpvine** | rises from pipes on a cycle — name reserved, not designed (YAGNI) |

Names v0.1 → v1.0: Mudlet → Grubbit, Clank → Dentshell, Snapvine → Gulpvine. Reason: a name clash
check found "Clank" too close to a well-known console character, "Mudlet" is an existing software
product and "Snapvine" a former company (§13, N). Pip-Wrench has no clash found. The vertical slice
uses only Grubbit and Dentshell (§10). Other roles of the study (§4 there) are not designed yet.

## 3.1 Screen

- Logical screen **256 × 240 px**, playfield of **16 × 16 px tiles**, 16 columns visible, 15 rows:
  2 HUD rows (screen y 0–31) then 13 level rows (screen y 32–239). Level row `r` is at screen
  `y = 32 + 16 r`, so ground rows 11–12 are at y 208–239. [gen] — fixes the coordinates used by §8.
- Integer-scaled to the window, letterboxed (viewport module from retro-shmup, ADR-0003). [gen]
- HUD: score, coins, level, timer. [gen]
- Sprites: Small hero 16 × 16, Big hero 16 × 32, enemies 16 × 16, shell 16 × 16. [gen] (§13, Q3)

## 4. Hero movement

All values are **fixed-point integers**, converted from the study (§1 there). Storage units [int]:

| Quantity | Stored as | Note |
|---|---|---|
| Position `x`, `y` | 1/256 px | on-screen pixel = value >> 8 |
| Horizontal speed `vx` | speed units of 1/16 px/frame, plus an 8-bit fraction `vxFrac` | caps 24 / 40 are in speed units; per frame `x += vx * 16` |
| Horizontal acceleration | added to `vxFrac` each frame; carry/borrow moves `vx` by 1 unit | table values 228 / 152 / 208 are in 1/256 speed unit = 1/4096 px/frame² |
| Vertical speed `vy` | 1/256 px/frame | −4 px/frame = −1024; terminal 4 px/frame = 1024 |
| Gravity | added to `vy` each frame | table values 32 … 144 are already in 1/256 px/frame² |

Hero state beyond position and speed (§4.3): `facing`, `runHoldFrames`, `gravityRow` (chosen at
take-off), `takeoffY`, `jumpWasHeld`, `invulnFrames`. Sub-pixel rounding differences against the
reference are checked by a golden-trajectory test (ADR-0005).

### 4.1 Horizontal [src] (main study §2, verified in the public disassembly)

| Rule | Value |
|---|---|
| Walk cap | 24 (1.5 px/frame) |
| Run cap (run button held, on ground, input = moving direction; kept 10 frames after release) | 40 (2.5 px/frame) |
| Run cap in the air | 40 if \|speed\| ≥ 25 at that frame, else 24 |
| Acceleration "strong" | 228 / 4096 px/frame² (used when running) |
| Acceleration "normal" | 152 / 4096 (walking, low speed) |
| Acceleration "fast-walk" | 208 / 4096 (walking with \|speed\| ≥ 33 or the running-speed flag) |
| Turnaround (facing ≠ moving) | acceleration × 2 (skid) |
| Skid end | holding the opposite direction with \|speed\| < 11 → speed 0, direction flips |
| Release on ground (no input) | decelerate with the same table, **clamp to 0** when crossing zero [gen] (bug-free, §11) |
| Release in the air | speed kept unchanged |
| Cap drop | speed above the current cap is clamped to the cap the same frame |

### 4.2 Jump and gravity [src] unless tagged

Take-off from the ground on a new press of the jump button (edge-triggered, complement §5).

| \|X speed\| at take-off | Initial Y speed | Gravity while held | Gravity after release / falling |
|---|---|---|---|
| 0–15 | −4 px/frame | 32 / 256 | 112 / 256 |
| 16–24 | −4 | 30 / 256 | 96 / 256 |
| ≥ 25 | −5 | 40 / 256 | 144 / 256 |

- Released early (after at least 1 px of rise) or vertical speed ≥ 0 → the falling gravity
  applies until landing.
- The gravity pair is kept until the next jump (stomp bounces do not change it).
- **Terminal fall speed: 4 px/frame, clean clamp** [gen] (ADR-0002 drops the reference's one-frame
  5 px/frame overshoot).
- Stomp bounce: Y speed −4 px/frame [src] (complement §6: shared by walkers and shelled walkers).
- No double jump, no wall jump, no air jump. [gen]

### 4.3 Hero states

Small → Big (growth item) → (shooter item: later levels). [src]

- Hit while Big → Small with **158 frames (≈ 2.6 s)** of invulnerability; the sprite blinks every
  4 frames (blink is a game-feel effect, toggleable, §12). [int] — midpoint of the reference's
  148–168 frames (complement §1); the 0–20 frame phase jitter is not reproduced (D1). (§13, Q1)
- Hit while Small → death. Falling into a pit → death. Timer at 0 → death. [src]
- A growth item picked up during invulnerability makes the hero Big and ends the invulnerability. [gen]
- Death hop: Y speed −4 px/frame. [src]
- Formalised in `docs/architecture/diagrams/gameplay/` (state diagram).

## 5. Collision

- Tile collision on a 16 × 16 grid, axis-separated: move X then resolve, move Y then resolve. [gen]
- Hitboxes [gen] (§13, H): **Small 12 × 14 px, Big 12 × 28 px**, horizontally centred on the sprite
  and flush with its feet. Enemies: Grubbit 12 × 12, Dentshell 12 × 14, shell 12 × 12, growth
  item 14 × 14. All hitboxes are data in one table, tunable without touching rules.
- Head bump: a block above the hero's centre is hit; vertical speed set to +1 px/frame. [src]
- Bumped block placement: start offset 4 px (Big) or 18 px (Small), chosen by hero size
  (complement §6). [src]
- Corner tolerance [gen]: a head bump within 4 px of a block edge nudges the hero aside instead
  of stopping the jump. Replaces the reference's inconsistent corner clips (§11).
- **Contact test every frame** [gen] (the reference tests every other frame, a hardware shortcut;
  §13, Q6).
- Hero–enemy contact is resolved after movement, in a fixed entity order (index order), so replays
  are deterministic. [gen]

## 6. Enemies

| Enemy | Speed | Rule | Stomp | Other kills |
|---|---|---|---|---|
| Grubbit | 0.5 px/frame [src] | walks; turns at walls; falls off ledges; turns when meeting another enemy | flattened, removed after 30 frames [gen] | bumped block below, shell |
| Dentshell | 0.5 px/frame [src] | as Grubbit | → shell, still | bumped block below, shell |
| Shell (kicked) | 3 px/frame [src] | slides, turns at walls, kills enemies in its path, hurts the hero | stops the shell | — |

- **Shell revival** [int] (§13, Q2): a still shell revives as a Dentshell after **280 frames**
  (≈ 4.7 s, a round value inside the reference's 231–336 frames [src]); during the last 60 frames it wobbles [gen].
  A kicked shell does not revive while it moves.
- Enemies are activated when their spawn column enters the screen + 1 column, removed when they
  leave 2 columns behind the camera. [gen]
- Touching an enemy from the side or below → hit (§4.3).
- "Stomp" = hero falling (Y speed > 0) and hero's feet above the enemy's centre. [gen]

## 7. Levels

- One-way horizontal scrolling [src]: the camera follows the hero once the hero's on-screen x
  reaches **80 px**; between 80 and 112 px a displacement of 2 px or more is reduced by 1 px, from
  112 px on the full displacement is used; the camera never scrolls back left, the hero is clamped
  at the left screen edge, and the scroll locks at the end of the level (complement §2, D2). This
  replaces the v0.1 wording "passes the screen centre" (Q14).
- Tiles [gen]: ground, hard block, brick (breaks when Big, bumps when Small), item block (coin,
  growth item), hidden coin block, pipe (2 tiles wide, not enterable in the slice), flag pole,
  goal house. Glyphs and behaviours: level study §2.
- **Level format** [gen] (§13, L): a text grid, one character per tile (glyphs in the level study),
  plus an entity list (spawn column, row, type), plus a header with the level timer (§8). Levels
  are data under `src/domain/level/`, never code.
- Level **1** of the vertical slice is **our own layout** (§10, level study §4), 200 columns long [gen].
- Layout rules E1–E7 (level study §3.3) are checkable predicates; a layout test (brick 9) enforces
  them for every level.

## 8. Items, scoring, lives and timer

Scoring values are tagged **[gen]**: they follow the commonly documented values of the reference
genre and were **not** re-verified in the disassembly for this revision (see §13, S for the
decision and the open verification item).

| Event | Points |
|---|---|
| Coin | 200 [gen] |
| Brick broken | 50 [gen] |
| Growth item collected | 1000 [gen] |
| Stomp chain without landing | 100, 200, 400, 500, 800, 1000, 2000, 4000, 5000, 8000, then +1 life [src] (main study §5) |
| Shell chain kills | same chain [gen] |
| Time bonus at the flag | 50 per remaining timer unit [gen] |

Flag pole (five bands [src]; mapping to our rows [int]) (§13, Q8): the grab row is the tile row
of the hero's feet when he touches the pole (rows 2–9 in the slice):

| Pole rows | Points |
|---|---|
| 2–3 | 5000 |
| 4–5 | 2000 |
| 6 | 800 |
| 7–8 | 400 |
| 9 | 100 |

The 5000 band must be reachable from the top step of the final staircase, and the 100 band is the
lowest grab (row 9, e.g. a low jump onto the pole base side): a golden-trajectory test checks both
(ADR-0005).

Lives and timer:

- Start with **3 lives** [gen]; 100 coins → +1 life [src]; no continue in the slice (§13, V).
- **Level timer in the level header**: 400 units for the slice [src for the unit], 1 unit =
  24 frames [src] (complement §1); the warning jingle plays at 100 units and the music tempo
  rises [src]; 0 → death [src]. Later levels may use 300 or 200 (Q5, Q11, §13).
- All timers (invulnerability, shell revival, level timer, pause lock) are counted in **frames** at
  60 Hz (complement D1). Pausing stops all timers except the 43-frame pause lock, which keeps counting while paused so the toggle can resume. [src]/[gen]
- Growth item: emerges from its block (1 px every 4th frame for 16 px, ≈ 64 frames) then slides
  right at 1 px/frame, turns at walls, falls off ledges. [src]

## 9. Game flow and meta

Title → Level intro (level name, lives, 2 s = 120 frames [gen]) → Playing ⇄ Paused → Dying →
Level intro / Game over → name entry (if top 10) → Hall of fame → Title.

End of level [src] (complement §6): flag touched → the hero slides down the pole (input locked) →
jumps off to the right and walks right on forced input → enters the goal house (drawn behind
him) → time-to-points tally (timer units × 50) → next level, or Title after the last level in the
slice. Formalised as state diagrams in `docs/architecture/diagrams/system/` and
`docs/architecture/diagrams/gameplay/`.

Pause [gen] (§13, Q7): Enter, P or the gamepad Start button toggle pause; Esc opens the menu. The
game **auto-pauses** when the page loses focus or becomes hidden. Pause is an input event: it is
recorded in the replay script and the simulation does not advance while paused. A 43-frame lock
prevents re-trigger [src]; it keeps counting during pause (§8). On touch, a pause button sits at the top
of the screen, and an auto-pause shows a "tap to resume" overlay [gen].

Controls [gen] (complement §5): keyboard arrows or A/D move, Space or W jump, Shift or X run;
gamepad D-pad or left stick (dead zone 0.4), bottom button jump, left button run. Left + right
together are ignored. Touch (D5, fixed pad, plus a pause button): landscape; left pad (left, right, down) bottom-left,
jump and run bottom-right; targets ≥ 48 px; multi-touch. Rebinding for keyboard and gamepad.

Meta features reused from retro-shmup (copy or shared package decided at the first code brick,
ADR-0003): input with rebinding, local save (options, top 10), name entry, EN/FR catalogue,
synthesised audio.

## 10. Vertical slice (first playable target)

One overworld level of our own, containing (counts as defined in the level study §4.3): ground
with 3 pits, **6 brick runs** (one brick row per run), 8 item blocks (6 coin, 2 growth items),
**1 hidden coin block** (extra, not in the 8), 4 pipes, 2 staircases (a pyramid and the final
climb), 12 Grubbits, 3 Dentshells, flag pole and goal house. The full layout is in the level
study §4.

Done when: the full flow Title → Level → Flag → Tally → Title works, the movement tables pass
their unit tests, and a scripted input run replays identically.

Pit 3 (5 tiles) is designed for a run but a walking jump is not excluded (§13, Q13); a
golden-trajectory test measures the real margin, and widening it by one tile is the recorded
fallback.

## 11. Reference bugs deliberately removed

| Reference defect | Our rule |
|---|---|
| Vertical cap check lets speed reach 5 px/frame | §4.2 — clean 4 px/frame clamp |
| No stop-at-zero clamp (sub-pixel oscillation at rest) | §4.1 — clamp to 0 |
| Wall / corner clips | §5 — axis-separated collision + corner tolerance |
| Contact test every other frame | §5 — every frame |
| Wrong-warp glitch levels | none (no warp zones in the slice) |
| Sprite-per-line flicker | none (canvas has no sprite limit) |
| PAL timing differences | fixed 60 Hz everywhere |
| Phase jitter of shared 21-frame timers | §8 — all timers in frames |

Kept on purpose because they are part of the feel: gravity pair kept until the next jump, cap drop
when the run timer expires, momentum kept in the air, the −1 px scroll quirk between 80 and 112 px.

## 12. Non-functional

- 60 fps on a 2019 mid-range phone; ≤ 2 ms of logic per step; no allocation in the loop. [gen]
- Deterministic: an input script replays the same run (ADR-0005).
- Accessibility [gen]: every game-feel effect (blink, screen shake, flashes) is toggleable and
  respects photosensitivity options; remappable input.
- Animation cadence [gen] (§13, Q3): walk cycle of 3 frames advancing every 8 frames at walk speed
  and every 5 at run speed; floating score rises 0.5 px/frame for 48 frames; constants live in one
  table, tuned at the assets brick (#6).

## 13. Decision matrices (MCDA)

Weights sum to 100, scores 0–5, totals /500; the owner validated the weights and the winners on
2026-10-19 (plan approval). Scores are design judgments, not measurements. Notation: criteria with
weights, then option totals; the winner is the solution, the runner-up is the recorded fallback.

| ID | Choice | Criteria (weight) | Totals | Fallback |
|---|---|---|---|---|
| N | Cast names | IP safety 40, memorability 20, EN/FR readability 20, rename cost 20 | **B rename the three: 440**, C rename only the shelled walker: 360, A keep v0.1 names: 300 | C (360) |
| H | Hitboxes | feel fidelity 30, fairness 25, level compatibility 20, simplicity 10, stomp/bump consistency 15 | **A 12×14 / 12×28: 430**, B 10 wide: 390, C 14×16 / 14×30: 360 | B (390) |
| Q1 | Hit invulnerability | fidelity 40, fairness 25, simplicity 15, photosensitivity exposure 20 | **A 158 frames: 460**, C 150 frames: 435, B 120 frames: 330 | C (435) |
| Q2 | Still shell | fidelity 30, fairness 25, simplicity 20, test cost 10, state risk 15 | **A revive at 280 frames + wobble: 410**, C two reference durations: 365, B stays still: 360 | C (365) |
| Q5/Q11 | Slice level timer | fidelity 25, tension 20, first-timer fairness 30, tally balance 15, simplicity 10 | **A 400: 395**, C 200: 375, B 300: 365 | C (375) |
| Q6 | Contact test | determinism/fairness 30, fidelity 20, cost 15, simplicity 20, tunnelling 15 | **A every frame: 445**, B every other frame: 370, C swept: 370 | B (370) |
| Q7 | Pause | run protection 30, replay 20, simplicity 20, browser reliability 15, mobile 15 | **A auto-pause on blur or hidden: 430**, C hidden only: 415, B none: 320 | C (415) |
| Q8 | Flag bands | fidelity 30, reachability/fairness 30, readability 20, testability 10, simplicity 10 | **C by tile rows: 440**, A reference cutoffs: 390, B equal pixel bands: 360 | A (390) |
| Q9 | "6 brick rows", "2 staircases" | consistency with validated layout 40, no re-layout 30, clarity 30 | **A keep the layout, reword §10: 470**, B literal rows: 300, C drop the quota: 260 | B (300) |
| Q10 | Hidden block | traceability 35, consistency with level D5 30, quota simplicity 20, minimal change 15 | **A add to §10: 465**, B unlisted extra: 365, C remove: 300 | B (365) |
| Q13 | Pit 3 | run intent 25, fairness/margin 30, no re-layout 20, determinism 15, rhythm 10 | **A keep 5 tiles: 435**, B 6 tiles: 380, C 7 tiles: 315 | B (380) |
| Q3 | Animation, floating score, sizes | closes #5 30, IP independence 20, cost 20, feel risk 15, reversibility 15 | **A our own values: 470**, B measure on video: 350, C defer: 335 | B (350) |
| V | Lives | fidelity 30, fairness 25, flow simplicity 25, challenge 20 | **A 3 lives: 450**, B 5 lives: 350, C 3 + continue: 325 | B (350) |
| L | Level format | diff-friendly 25, testability 25, simplicity 20, tooling 15, reuse 15 | **A text grid + entity list: 440**, B JSON: 370, C TMX: 280 | B (370) |
| S | Scoring values | fidelity 35, verifiability 20, simplicity 20, balance 25 | **A reference values: 455**, B own round scale: 370, C reference chain only: 340 | B (370) |

Sensitivity notes:

- **Q5/Q11** is narrow: moving 7 weight points from fairness to tension makes C (200 units)
  overtake A. The timer is a per-level header field, so the change is a data edit.
- **Q7**: A leads C by 15; raising "browser reliability" by 8 points or more makes C first.
- **Q6**: B and C tie as runner-up; B is the drop-in (one flag).
- **N** is decided on IP safety; the gap is large enough that no weight shift flips it.

Closed without a new matrix (decision already validated elsewhere): Q14 camera trigger (complement
D2, 480/500, wording aligned in §7), Q4 (withdrawn in the complement), fixed-point storage units
(project rule "deterministic 60 Hz"), touch scheme (complement D5, 420/500), terminal fall speed
(ADR-0002). **Q12** does not exist in either annex (the numbering skips from Q11 to Q13).

**Open verification item (S):** the coin, brick and growth-item values and the time bonus are
tagged [gen] because the public disassembly was not re-read for this revision; they are checked
against the disassembly at the first code brick, and any difference is a one-line table edit.
The rest of the scoring (chain, five flag bands) is [src].

## Change log

| Version | Date | Change |
|---|---|---|
| 0.1 | 2026-10-16 | Brick 0 draft: vision, IP boundary, cast, movement tables, collision, enemies, scoring, flow, slice, bugs removed. |
| 1.0 | 2026-10-19 | Every *(proposed)* GDD item settled (§13). Cast renamed (N). Hitboxes, enemy hitboxes, sprite sizes. Hit invulnerability 158 frames (Q1). Shell revival (Q2). Timer in the level header, 400 for the slice (Q5/Q11). Contact test every frame (Q6). Pause and auto-pause (Q7). Flag bands by tile rows (Q8). §10 counts reworded (Q9, Q10). Pit 3 kept (Q13). Camera trigger aligned to 80/112 px (Q14). Animation cadence (Q3). Lives (V), level format (L), scoring (S). Screen rows fixed (§3.1). End-of-level sequence added to §9. Tags [src]/[int]/[gen] on every figure. |

## Decision record

| Date | Decision | Ref |
|---|---|---|
| 2026-10-16 | Side-scroller, codename retro-plumber, study only until retro-shmup's slice | owner, ADR-0004 |
| 2026-10-16 | Reference = the 1985 home-console side-scroller (EU release 15 May 1987); slice first | owner |
| 2026-10-16 | Physics tables verified against the disassembly (study v0.2) | study |
| 2026-10-19 | Complement and level studies accepted; their choices stand (D1–D5, level D2–D5) | owner |
| 2026-10-19 | GDD v1.0: plan, weights and winners of §13 validated | owner |
