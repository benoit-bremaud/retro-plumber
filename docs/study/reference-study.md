# Reference study — the 1985 home-console side-scroller

Status: **draft v0.2** (brick 0, 2026-10-16 — physics values checked against the disassembly). Input for the GDD, not a design contract.

This study describes the **behaviour** of the reference game (the first side-scrolling
platformer of its series, released in Europe on 15 May 1987) so that retro-plumber can
reproduce its *feel* with our own character, levels, art and sound. Per the project's IP rule,
no character, enemy, item or level name of the reference game is used below; roles are named
generically.

Every value carries a confidence tag:

- **[src]** — read in a cited public source.
- **[int]** — our interpretation of a sourced value (unit conversion, meaning of a table index);
  to be confirmed against the community disassembly before it enters the GDD.
- **[gen]** — common genre knowledge, not yet checked against a source.

## 1. Simulation frame

| Topic | Value | Tag |
|---|---|---|
| Logic rate | 1 update per video frame (60 Hz NTSC, 50 Hz PAL) | [gen] |
| Screen | 256 × 240 px, playfield built from 16 × 16 px metatiles (16 columns × 15 rows) | [gen] |
| Horizontal speed unit | 1/16 px per frame (signed byte; high nibble = px, low nibble = 1/16 px) | [src] |
| Horizontal acceleration accumulator | 1/256 of a speed unit = 1/4096 px/frame²; carry feeds the speed | [src] |
| Vertical speed unit | whole px per frame (signed byte) | [src] |
| Vertical acceleration accumulator | 1/256 px/frame²; carry feeds the speed, a separate 1/256 px accumulator carries sub-pixel position | [src] |

Design consequence: our domain uses **fixed-point integers** (no floats) so the simulation is
deterministic and matches the reference granularity (see CLAUDE.md, fixed 60 Hz timestep).

## 2. Horizontal movement [src]

Units: speed in 1/16 px/frame.

| Rule | Value |
|---|---|
| Walk soft cap (B not held) | ±24 → 1.5 px/frame |
| Run hard cap (run timer active) | ±40 → 2.5 px/frame |
| Walk / run ratio | 3/5 |
| Run timer | set to 10 while B held, on ground, input direction = movement direction; −1 per frame otherwise |
| "Running speed" flag | set when \|speed\| ≥ 28 |
| Water caps | hard ±24, soft ±16 on ground |
| Pipe-entry cutscene cap | 12 |
| Double acceleration | when facing ≠ movement direction (turnaround / skid): friction value doubled |
| Speed resets to 0 | wall collision, death, screen transition; and \|speed\| < 11 with a non-A button pressed |

Acceleration table — **verified in the disassembly** (`FrictionData`, `PlayerPhysicsSub`,
`ImposeFriction`). One value is used both to accelerate (direction held) and to decelerate (no
direction, on the ground). It is added to the 1/256 accumulator, so px/frame² = value / 4096.

| Index | Player 1 | px/frame² | Player 2 (alternate feel, from the community documentation) |
|---|---|---|---|
| 0 | 0xE4 | 0.0557 | 0xB4 |
| 1 | 0x98 | 0.0371 | 0x68 |
| 2 | 0xD0 | 0.0508 | 0xA0 |

Facing ≠ moving direction (turnaround, skid): the value is doubled (`asl`/`rol`).

Index and speed cap per situation (verified, `PlayerPhysicsSub`):

| Situation | Speed cap | Accel index |
|---|---|---|
| Ground, input = moving direction, B held (run timer set to 10) | 40 | 0 |
| Ground, same, B released but run timer not yet 0 | 40 | 0 |
| Ground, otherwise (walking, turning, no input) | 24 | 1, or 2 if running-speed flag or \|speed\| ≥ 33 |
| Ground in a water area | 16 | 1 or 2 (same rule) |
| Air, \|speed\| ≥ 25 at the time | 40 | 0 |
| Air, \|speed\| < 25 | 24 | 1 or 2 (same rule) |

- **The cap is a hard clamp, not a slow-down:** when the cap drops from 40 to 24 (run timer
  expired while still holding the direction), the speed snaps to 24 on the next frame.
- **Air control:** in the air the table is applied only while a direction is held; with no input,
  horizontal speed is kept unchanged. On the ground, no input decelerates with the same table.
- **Skid end (verified, `GetPlayerAnimSpeed` → `ProcSkid`):** while the held direction is
  opposite to the moving direction and \|speed\| < 11, the speed and accumulator are set to 0 and
  the moving direction flips — the turnaround completes instantly from there.
- **Release with no input (read in `ImposeFriction`, medium confidence):** there is no explicit
  stop-at-zero clamp. Friction pushes the speed toward 0 from whichever side it is on and stops
  acting only when the speed is exactly 0, so the speed may cross zero and settle in a few frames
  through tiny sub-pixel oscillations. Our design will use a clean clamp to 0 (GDD §4).

## 3. Vertical movement [src]

Jump initial vertical speed, indexed by horizontal speed at take-off:

| \|X speed\| | Index | Initial Y speed |
|---|---|---|
| < 9 | 0 | −4 px/frame (0xFC) |
| 9–15 | 1 | −4 |
| 16–24 | 2 | −4 |
| 25–27 | 3 | −5 (0xFB) |
| ≥ 28 | 4 | −5 |

Gravity (added per frame to the 1/256 accumulator, so px/frame² = value / 256) — **verified**
(`JumpMForceData`, `FallMForceData`, `ImposeGravity`); the take-off speed thresholds 9 / 16 / 25 /
28 are also verified (`ProcJumping`):

| Index | Rising (jump held) | Falling / jump released |
|---|---|---|
| 0–1 (standing) | 0x20 → 0.125 | 0x70 → 0.4375 |
| 2 (walking) | 0x1E → 0.117 | 0x60 → 0.375 |
| 3–4 (running) | 0x28 → 0.156 | 0x90 → 0.5625 |
| 5 (swimming) | 0x0D | 0x0A |

- **Variable jump height (verified, `JumpSwimSub`):** while airborne, the falling gravity
  replaces the rising one as soon as the vertical speed becomes ≥ 0, **or** the jump button is no
  longer held once the player has risen at least 1 px above the take-off height. Releasing early =
  short hop. This is the single most important feel rule.
- **Gravity state persists** until the next jump: stomping an enemy or bouncing on a spring does
  not change it; the lowest gravity applies at level start and after a pipe exit.
- **Terminal fall speed (verified, `MovePlayerVertically` → `ImposeGravity`):** the intended cap
  is 4 px/frame. The clamp only fires when the speed is ≥ 4 **and** the accumulator is ≥ 0x80, so
  with the strong falling gravities the speed regularly reaches 5 px/frame for one frame before
  being clamped back to 4. This explains the ~4.2 px/frame average measured by TAS players.

Vertical speed changes: spring bounce −12 (jump held) / −7; enemy stomp bounce −3 or −4; death
hop −4; bumping a hard block or platform from below sets +1; breaking a soft block −2.

## 4. Cast and objects (generic roles) [gen]

| Role | Behaviour to reproduce | Defeated by |
|---|---|---|
| Walker | walks straight, turns at walls, falls off ledges | stomp, hit from below, projectile, shell |
| Shelled walker | walks; stomp → shell; kicked shell slides and chains kills | stomp, shell |
| Winged shelled walker | hops or flies on a fixed path; stomp removes wings | stomp ×2 |
| Pipe plant | rises from pipes on a timer; stays hidden when the player is next to the pipe | projectile |
| Thrower | floats above, drops spiky crawlers | stomp, projectile |
| Hammer thrower | jumps between platforms, throws arcs | projectile, hit from below |
| Bullet launcher | fires straight projectiles | stomp on projectile |
| Water swimmers / leapers | sine or arcing paths in water / bridges | projectile |
| End-of-castle boss | patrols a bridge, breathes fire | bridge axe / projectiles |

Power-up chain: small → big (growth item) → shooter (fire item); hit while powered → shrink to
small with a short invincibility. Star item: temporary invincibility. Extra-life item, 100 coins
→ extra life.

## 5. Level grammar [gen]

- 8 worlds × 4 levels; overworld, underground, underwater, castle themes.
- One-way horizontal scrolling (the camera never scrolls back left).
- Blocks: brick (breakable when big), question block (coin / power-up), hidden block, multi-coin
  brick, hard block, pipes (some enterable), moving and falling platforms, springs.
- Level end: flag pole (height-based score) then castle; castle levels end at the boss bridge.
- Timer: 400 units (faster than seconds), bonus for remaining time at the flag.
- Score chain on consecutive stomps without touching the ground: 100 → 200 → 400 → 500 → 800 →
  1000 → 2000 → 4000 → 5000 → 8000 → extra life.

## 6. Known reference bugs and quirks (candidates to drop)

Same policy as retro-maze (faithful rules, without the original bugs) — decided in ADR-0002.

- Flawed vertical-speed cap (§3).
- Wall clips / corner clips and the wrong-warp glitch level.
- 3-objects-per-column sprite limit and flicker.
- Arbitrary-code-execution and RAM-dependent tricks.

## 7. Open questions for the GDD

1. Stop-at-zero with no input (§2): clean clamp to 0 instead of the sub-pixel oscillation.
2. Our terminal fall speed: clean 4 px/frame cap (drops the 5 px/frame overshoot), or keep the
   overshoot for an exact feel? Decided in ADR-0002: clean clamp.
3. Keep the second player's "alternate feel" table as a selectable option, or one feel only?
4. Fixed 60 Hz for every region (yes, recommended — we are not emulating PAL).
5. Vertical slice content: one overworld level with walker + shelled walker, bricks, question
   blocks, pipes, growth item, flag pole.

## Sources

- SMBpedia — Movement: <https://simplistic6502.github.io/smb1_tll/smbpedia_movement.html>
- Speedrun.com — General game mechanics guide: <https://www.speedrun.com/smb1/guides/pbl9d>
- TASVideos — "minimum A presses" submission (gravity states): <https://tasvideos.org/7094S>
- TASVideos — SMB3 resources (subpixel convention, used for comparison only):
  <https://tasvideos.org/GameResources/NES/SuperMarioBros3>
- Release date (EU 15 May 1987): <https://www.mariowiki.com/Super_Mario_Bros.>
- doppelganger, "SMBDIS.ASM — a comprehensive disassembly" (2012), mirror:
  <https://gist.github.com/1wErt3r/4048722> — routines `PlayerPhysicsSub`, `ImposeFriction`,
  `JumpSwimSub`, `ProcJumping`, `GetPlayerAnimSpeed`, `ProcSkid`, `MoveObjectHorizontally`,
  `MovePlayerVertically`, `ImposeGravity`, and the tables `FrictionData`, `JumpMForceData`,
  `FallMForceData`. Read for
  verification only; no code or data from it is copied into this project.
- SourceGen port of the same disassembly: <https://6502disassembly.com/nes-smb/>.
