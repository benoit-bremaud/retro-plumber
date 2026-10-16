# retro-plumber — Game Design Document

**Version:** 0.1 — 2026-10-16 (brick 0, design study).
**Status:** draft for owner review. Items marked *(proposed)* await owner validation.

This document is the design contract: no rule is implemented before it is written here, and the
code follows these tables. The behavioural evidence behind the numbers is in
[`docs/study/reference-study.md`](../study/reference-study.md); technical choices live in the
ADRs under [`docs/decisions/`](../decisions/); the UML study under
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
4. **Deterministic and testable.** Fixed 60 Hz simulation in fixed-point integers and a seeded
   RNG: the same inputs replay the same run (CLAUDE.md, ADR-0005).
5. **Clean architecture.** The rules below live in the pure domain; the browser is an adapter
   (CLAUDE.md § Architecture).

## 2. Behavioural reference and IP boundary

The behavioural evidence is the community documentation (SMBpedia) and doppelganger's public
disassembly, read for verification only (study § Sources). Nothing from them is copied as code or
data: tables are re-expressed in our own units and structures.

- **Reproduced:** movement rules, speed and gravity values, timings, scoring rules, the general
  grammar of blocks and enemies.
- **Never reproduced:** title, logo, character and enemy names, sprites, tiles, sounds, music,
  level layouts, the castle / princess story. Cast (§3) and levels (§7) are original.

## 3. Cast *(proposed — names and look are ours)*

| Role | Name | Behaviour (§6) |
|---|---|---|
| Hero — a plumber-engineer | **Pip-Wrench** ("Wrench") | §4 |
| Walker | **Mudlet** | walks, turns at walls, falls off ledges |
| Shelled walker | **Clank** | walks; stomp → shell; kicked shell slides |
| Pipe plant | **Snapvine** | rises from pipes on a cycle |

The vertical slice uses only the walker and the shelled walker (§10). Other roles of the study
(§4 there) are listed for later levels, not designed yet (YAGNI).

## 3.1 Screen

- Logical screen **256 × 240 px**, playfield of **16 × 16 px tiles**, 16 columns visible, 13 rows
  of level + 2 rows of HUD.
- Integer-scaled to the window, letterboxed (viewport module from retro-shmup, ADR-0003).
- HUD *(ours)*: score, coins, level, timer.

## 4. Hero movement

All values in **fixed-point integers**. Horizontal speed in 1/16 px per frame; accelerations in
1/4096 px/frame²; vertical speed in 1/256 px per frame internally (study §1). Values below are
taken from the study and **converted to our units**.

### 4.1 Horizontal

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
| Release on ground (no input) | decelerate with the same table, **clamp to 0** when crossing zero *(bug-free, §11)* |
| Release in the air | speed kept unchanged |
| Cap drop | speed above the current cap is clamped to the cap the same frame |

### 4.2 Jump and gravity

Take-off from the ground on a new press of the jump button.

| \|X speed\| at take-off | Initial Y speed | Gravity while held | Gravity after release / falling |
|---|---|---|---|
| 0–15 | −4 px/frame | 32 / 256 | 112 / 256 |
| 16–24 | −4 | 30 / 256 | 96 / 256 |
| ≥ 25 | −5 | 40 / 256 | 144 / 256 |

- Released early (after at least 1 px of rise) or vertical speed ≥ 0 → the falling gravity
  applies until landing.
- The gravity pair is kept until the next jump (stomp bounces do not change it).
- **Terminal fall speed: 4 px/frame, clean clamp** *(proposed, ADR-0002: drops the reference's
  one-frame 5 px/frame overshoot)*.
- Stomp bounce: Y speed −4 px/frame *(proposed: one value instead of the reference's −3/−4)*.
- No double jump, no wall jump, no air jump (genre rule).

### 4.3 Hero states

Small → Big (growth item) → (shooter item: later levels). Hit while Big → Small with 2 s of
invulnerability *(proposed)*. Hit while Small → death. Falling into a pit → death.
Formalised in `docs/architecture/diagrams/gameplay/` (state diagram).

## 5. Collision

- Tile collision on a 16 × 16 grid, axis-separated: move X then resolve, move Y then resolve.
- Hitbox *(proposed)*: Small 12 × 14 px, Big 12 × 28 px, centred on the sprite's feet.
- Head bump: a block above the hero's centre is hit; vertical speed set to +1 px/frame.
- Corner tolerance *(ours)*: a head bump within 4 px of a block edge nudges the hero aside instead
  of stopping the jump. Replaces the reference's inconsistent corner clips (§11).

## 6. Enemies

| Enemy | Speed | Rule | Stomp | Other kills |
|---|---|---|---|---|
| Mudlet | 0.5 px/frame *(proposed)* | walks; turns at walls; falls off ledges; turns when meeting another enemy | flattened, removed after 0.5 s | bumped block below, shell |
| Clank | 0.5 px/frame | as Mudlet | → shell, still | bumped block below, shell |
| Clank shell (kicked) | 3 px/frame *(proposed)* | slides, turns at walls, kills enemies in its path, hurts the hero | stops the shell | — |

- Enemies are activated when their spawn column enters the screen + 1 column, removed when they
  leave 2 columns behind the camera *(ours)*.
- Touching an enemy from the side or below → hit (§4.3).
- "Stomp" = hero falling (Y speed > 0) and hero's feet above the enemy's centre.

## 7. Levels

- One-way horizontal scrolling: the camera follows the hero once the hero passes the screen
  centre and never scrolls back left.
- Tiles: ground, hard block, brick (breaks when Big, bumps when Small), item block (coin,
  growth item), hidden coin block, pipe (2 tiles wide, not enterable in the slice), flag pole,
  goal house.
- Our level format *(proposed)*: a text grid, one character per tile, plus an entity list
  (spawn column, row, type). Levels are data under `src/domain/level/`, never code.
- Level **1** of the vertical slice is **our own layout** (§10), ~200 columns long.

## 8. Items, scoring and lives

| Event | Points |
|---|---|
| Coin | 200 |
| Brick broken | 50 |
| Growth item collected | 1000 |
| Stomp chain without landing | 100, 200, 400, 500, 800, 1000, 2000, 4000, 5000, 8000, then +1 life |
| Shell chain kills | same chain |
| Flag pole | 100 – 5000 by height (5 bands) |
| Time bonus at the flag | 50 per remaining timer unit |

- 100 coins → +1 life. Start with 3 lives.
- Timer: 400 units, 1 unit = 24 frames *(reference ≈ 0.4 s per unit)*; warning sound at 100;
  0 → death.
- Growth item: emerges from its block, slides right at 1 px/frame, turns at walls, falls off
  ledges.

## 9. Game flow and meta

Title → Level intro (level name, lives, 2 s) → Playing ⇄ Paused → Dying → Level intro / Game
over → name entry (if top 10) → Hall of fame → Title. Flag → tally → next level (or Title after
the last level in the slice). Formalised as a state diagram in
`docs/architecture/diagrams/system/`.

Meta features copied from retro-shmup (ADR-0003): keyboard / gamepad / touch input with
rebinding, local save (options, top 10), name entry, EN/FR catalogue, synthesised audio.

Touch input *(proposed)*: on-screen left/right pad + jump + run buttons.

## 10. Vertical slice (first playable target)

One overworld level of our own, containing: ground with 3 pits, 6 brick rows, 8 item blocks
(6 coins, 2 growth items), 4 pipes, 2 staircases, 12 Mudlets, 3 Clanks, flag pole and goal house.
Done when: the full flow Title → Level → Flag → Tally → Title works, the movement tables pass
their unit tests, and a scripted input run replays identically.

## 11. Reference bugs deliberately removed

| Reference defect | Our rule |
|---|---|
| Vertical cap check lets speed reach 5 px/frame | §4.2 — clean 4 px/frame clamp |
| No stop-at-zero clamp (sub-pixel oscillation at rest) | §4.1 — clamp to 0 |
| Wall / corner clips | §5 — axis-separated collision + corner tolerance |
| Wrong warps, "minus" level | none (no warp zones in the slice) |
| Sprite-per-line flicker | none (canvas has no sprite limit) |
| PAL timing differences | fixed 60 Hz everywhere |

Kept on purpose because they are part of the feel: gravity pair kept until the next jump, cap drop
when the run timer expires, momentum kept in the air.

## 12. Non-functional

- 60 fps on a 2019 mid-range phone; ≤ 2 ms of logic per step; no allocation in the loop.
- Deterministic: a seed and an input script replay the same run (ADR-0005).
- Accessibility: reduced flash, remappable input, toggleable screen shake *(ours)*.

## Decision record

| Date | Decision | Ref |
|---|---|---|
| 2026-10-16 | Side-scroller, codename retro-plumber, study only until retro-shmup's slice | owner, ADR-0004 |
| 2026-10-16 | Reference = the 1985 home-console side-scroller (EU release 15 May 1987); slice first | owner |
| 2026-10-16 | Physics tables verified against the disassembly (study v0.2) | study |
