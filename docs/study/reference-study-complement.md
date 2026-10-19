# Reference study — complement (timing, camera, rendering, audio, controls, entities)

Status: **Accepted v0.3** (owner, 2026-10-19; brick 1, issue #3). Companion of
[`reference-study.md`](reference-study.md) (v0.2, hero physics). Input for the GDD, not a design
contract. Owner decision 2026-10-19: one annex file (MCDA 440 / 500 vs six files 420, see §9).

Same IP rule and tag legend as the main study: generic role names only; **[src]** read in a cited
public source, **[int]** our interpretation of a sourced value, **[gen]** genre knowledge or our own
proposal, not sourced. Values marked *(disasm)* were read in doppelganger's public disassembly
(read only, nothing copied; routine and table names are cited so the reader can check). Units:
speeds in 1/16 px/frame as in the main study (`0xF8` = −8 = −0.5 px/frame).

## 1. Timing and frame rate

| Topic | Value | Tag |
|---|---|---|
| Console video rate | NTSC ≈ 60.10 Hz, PAL ≈ 50.01 Hz; APU frame counter 60 / 50 Hz | [src] nesdev Cycle reference chart |
| Logic step | one game-engine pass per video frame; **no delta time** | [src] *(disasm, main loop)* |
| Our rate | fixed 60 Hz accumulator, every region (study §7 Q4, CLAUDE.md) | [gen] |
| Frame timers | decremented every frame (21 slots) | [src] *(disasm, `DecTimers`)* |
| Interval timers | decremented once every **21 frames** (control reloaded to 20, expires at −1) | [src] *(disasm, `IntervalTimerControl`)* |
| Level timer | 1 unit = **24 frames** (`GameTimerCtrlTimer` reload `0x18`); starts at 400, 300 or 200 depending on the level header (`GameTimerData`) | [src] *(disasm, `RunGameTimer`)* |
| Time-low warning | when the timer reads 100: warning jingle, then the level music continues at a faster tempo (note-length table offset +8) | [src] *(disasm, `RunGameTimer`, `MusicHandler`)* |
| Timer reaches 0 | death, as if hit while small | [src] *(disasm, `TimeUpOn`)* |
| Pause | start button; a 43-frame lock (`0x2B`) prevents re-trigger; pausing also stops all timers | [src] *(disasm, `PauseRoutine`)* |
| Hit invulnerability | `InjuryTimer` = 8 interval units = **148 to 168 frames** (2.5 to 2.8 s; the first unit is shortened by the phase of the shared 21-frame counter) | [src]+[int] |
| Star invulnerability | `StarInvincibleTimer` = `0x23` = 35 interval units = **714 to 735 frames** (about 11.9 to 12.25 s, same phase effect); palette cycles every 2nd frame, then every 8th when 8 units remain or fewer | [src]+[int] |
| Player–enemy contact test | run **every other frame** (frame-counter bit 0) | [src] *(disasm, `PlayerEnemyCollision`)* |

Consequences:

- GDD §8 timer (24 frames per unit, 400 units) is **confirmed**.
- GDD §4.3 hit invulnerability of **2 s is proposed, the reference gives about 2.5 to 2.8 s** → open question Q1.
- The every-other-frame contact test is a hardware-era shortcut, not a feel rule → drop it (Q6).

**Choice D1 — timer representation.** A: every timer in frames at 60 Hz (reference intervals × 21);
B: two tiers like the reference (frame + 21-frame interval); C: wall-clock milliseconds.

| Criterion | W | A Frames | B Two tiers | C Milliseconds |
|---|---|---|---|---|
| Determinism / replay | 30 | 5 — integers only | 5 | 1 — depends on the clock |
| Fidelity to reference durations | 25 | 4 — 21 × n, within one 21-frame phase | 5 — exact, phase included | 3 — rounding |
| Simplicity (KISS) | 20 | 5 — one unit | 3 — two counters | 3 |
| Testability (scripted runs) | 15 | 5 | 4 | 1 |
| Pause-safe / accessibility | 10 | 4 | 4 | 3 |
| **Total / 500** | 100 | **465** | 435 | 210 |

Points: A = 150+100+100+75+40; B = 150+125+60+60+40; C = 30+75+60+15+30. Sensitivity: with KISS
at 0, A 365 vs B 375, so B wins; A leads only while KISS weighs more than about 15. **Fallback:** B (435), drop-in (same durations, extra counter).
Decision: all timers in frames (reference intervals converted ×21, the phase jitter of 0 to 20 frames is not reproduced).

## 2. Camera and scrolling

| Rule | Value | Tag |
|---|---|---|
| Direction | horizontal only; never scrolls back left; player is clamped at the left edge of the screen | [src] *(disasm, `ScrollHandler`, `ChkPOffscr`)* |
| Scroll trigger | hero's on-screen x ≥ 80 px (`0x50`) | [src] *(disasm)* |
| Scroll amount | hero's frame displacement; between 80 and 112 px (`0x70`) a displacement of 2 px or more is reduced by 1 px, a displacement of exactly 1 px is kept; the full displacement from 112 px on | [src] *(disasm)* |
| Scroll lock | a flag stops scrolling at the level end, and while a boss or an end-of-level object is on screen | [src] *(disasm, `ScrollLock`)* |
| Scroll stops after side collision | not while the side-collision timer is running | [src] *(disasm)* |
| Platform riding | a moving platform's displacement is added to the scroll force | [src] *(disasm, `Platform_X_Scroll`)* |
| Screen width | 256 px; the right boundary is left + 255 | [src] *(disasm, `GetScreenPosition`)* |
| Right clamp | hero cannot leave the screen on the right either (16 px margin) and his speed is zeroed when pushing against the edge | [src] *(disasm, `ChkPOffscr`, `X_SubtracterData`)* |

**Choice D2 — camera model.** A: reference rules (threshold 80 / 112, one-way, integer scroll);
B: smoothed follow camera; C: reference rules plus a dead-zone smoothing of the scroll speed.

| Criterion | W | A Reference | B Smooth follow | C Reference + smoothing |
|---|---|---|---|---|
| Feel fidelity | 35 | 5 | 2 — changes level readability | 4 |
| Determinism | 20 | 5 — integers | 3 — fixed-point lerp | 4 |
| Simplicity | 20 | 5 | 2 | 3 |
| Level-grammar compatibility (one-way) | 15 | 5 | 3 | 5 |
| Small-screen readability (phone) | 10 | 3 — fixed 256 px | 4 | 4 |
| **Total / 500** | 100 | **480** | 255 | 395 |

Points: A = 175+100+100+75+30; B = 70+60+40+45+40; C = 140+80+60+75+40. **Fallback:** C (395), a
drop-in change confined to the scroll module. Decision: reference rules; the "−1 px between 80 and
112 for displacements of 2 px or more" quirk is a feel rule, kept.

## 3. Rendering

| Topic | Value | Tag |
|---|---|---|
| Resolution | 256 × 240 px; usable height 224 rows (player below row 224 is out of play) | [src] *(disasm, `CheckPlayerVertical`)*; [gen] |
| Tile model | 16 × 16 metatiles built from 8 × 8 tiles | [gen] |
| Palettes | master of 64 colours; 4 background + 4 sprite sub-palettes of 4 entries; entry 0 transparent, shared backdrop | [src] nesdev PPU palettes |
| Sprite size | hero and enemies drawn as 8 × 16 or 8 × 8 pieces; a big hero is 16 × 32, a small one 16 × 16 | [gen] |
| Layers | background (tiles) → sprites; a per-sprite priority bit puts a sprite **behind** the background (used for the hero entering a castle and for objects emerging from blocks) | [src] *(disasm, `PlayerBGPriorityData`, `GrowThePowerUp`)* |
| Colour animation | one background colour is rotated by a frame counter (blocks that pulse); star state cycles the hero's palette | [src] *(disasm, `ColorRotation`, `CyclePlayerPalette`)* |
| Animation timing | hero walk cycle advances every N frames, N taken from speed (`PlayerAnimTimerSet`); exact table to be verified | [int], open (Q3) |
| Sprite limits | 64 sprites, 8 per scanline → flicker; dropped (study §6) | [gen] |

**Choice D3 — palette and art constraint.** A: our own fixed palette (≤ 64 colours), sprites ≤ 3
colours + transparent per sub-palette; B: unrestricted truecolour; C: the console's master palette.

| Criterion | W | A Own fixed palette | B Truecolour | C Console palette |
|---|---|---|---|---|
| IP safety (ADR-0004) | 30 | 5 — own colours | 5 | 2 — evokes the reference look |
| Coherent retro look | 25 | 5 | 2 | 5 |
| Art production cost | 15 | 3 | 4 | 3 |
| Performance (pre-rendered, no allocation) | 15 | 5 | 4 | 5 |
| Contrast tuning / accessibility | 15 | 4 | 5 | 2 |
| **Total / 500** | 100 | **455** | 395 | 335 |

Points: A = 150+125+45+75+60; B = 150+50+60+60+75; C = 60+125+45+75+30. **Fallback:** B (395); a
change of art assets only, no code. Decision: own fixed palette; the per-sprite colour limit is an
art guideline checked by the asset linter at the assets brick (#6, not here).

## 4. Audio

| Topic | Value | Tag |
|---|---|---|
| Hardware voices | 2 pulse, 1 triangle (no volume control), 1 noise, 1 sample (DMC) | [src] nesdev APU |
| Voice use in the reference | music on pulse 2 (lead), pulse 1 (second voice), triangle (bass), noise (percussion); sound effects on pulse 1, pulse 2 and noise | [src] *(disasm, `Square1SfxHandler`, `Square2SfxHandler`, `NoiseSfxHandler`)* |
| Effect set (roles) | small jump, big jump, stomp, bump, projectile, shrink/pipe, enemy smack, flag pole, coin, power-up appears, power-up taken, extra life, vine, timer tick at the tally, block shatter, boss fire, boss fall | [src] *(disasm, `Sfx_*` constants)* |
| Effect priority | effects go through three **queues** (pulse 1, pulse 2, noise); a new request is honoured by bit order and by what is already playing | [src] *(disasm)* |
| Music kinds | area loops (overworld, underground, water, castle, cloud, star), event jingles (death, game over, level end, castle end, victory, time low) | [src] *(disasm, `*Music` constants)* |
| Music vs effects | death music silences both pulse effects; event music saves the area track and **restores it afterwards** | [src] *(disasm, `LoadEventMusic`, `AreaMusicBuffer_Alt`)* |
| Time-low tempo | the same music with all note lengths shortened | [src] *(disasm, `NoteLengthTblAdder`)* |
| Sequencing clock | notes are counted in frames (length tables of 1–54 frames) | [src] *(disasm, `MusicLengthLookupTbl`)* |

Everything heard is **ours**: no melody, rhythm, jingle or effect of the reference is transcribed
(ADR-0004). Only structure is reused: voice roles, queues, restore-after-jingle, tempo-up.

**Choice D4 — audio engine.** A: synthesised voices (Web Audio) modelled on 2 pulse + triangle +
noise, sequencer on the 60 Hz tick; B: pre-rendered audio files; C: A plus sampled hits.

| Criterion | W | A Synth voices | B Files | C Synth + samples |
|---|---|---|---|---|
| IP safety, own music | 25 | 5 | 4 | 5 |
| Tick-locked sequencing (replay-friendly) | 20 | 5 | 2 | 4 |
| Reuse of retro-shmup synth | 20 | 5 | 1 | 3 |
| Size / loading | 15 | 5 | 2 | 4 |
| Mobile latency and autoplay rules | 10 | 3 | 4 | 3 |
| Authoring effort | 10 | 3 | 3 | 2 |
| **Total / 500** | 100 | **460** | 260 | 375 |

Points: A = 125+100+100+75+30+30; B = 100+40+20+30+40+30; C = 125+80+60+60+30+20. Sensitivity:
reuse weight to 0 and size to 25 → A 410 vs C 355 (A still first). **Fallback:** C (375), additive
(the voice engine stays, samples are an extra layer behind `AudioPort`). Decision: synth voices.
The DMC channel is not modelled (YAGNI, no effect needs it).

## 5. Controls

| Topic | Value | Tag |
|---|---|---|
| Reference input | D-pad (left, right, down, up), two action buttons (jump, run / action), start, select | [src] *(disasm, `A_B_Buttons`, `Left_Right_Buttons`)* |
| Edge-triggered jump | a new jump needs a fresh press (previous-frame A/B saved each frame) | [src] *(disasm, `PreviousA_B_Buttons`)* |
| Down | crouch when big, pipe entry | [src] *(disasm, `PlayerMovementSubs`)* |
| Left + right together | not allowed on the console pad; our rule: ignore both | [gen] |
| Keyboard *(proposed)* | arrows or A/D move, Space or W jump, Shift or X run, Enter / P pause, Esc menu | [gen] |
| Gamepad *(proposed)* | D-pad or left stick (dead zone 0.4) move; bottom button jump; left button run; Start pause | [gen] |
| Touch *(proposed)* | landscape; left pad (left, right, down) bottom-left; jump and run buttons bottom-right; ≥ 48 px targets; multi-touch | [gen] |
| Rebinding | keyboard and gamepad rebindable, stored through `StoragePort` (reuse from retro-shmup, ADR-0003) | [gen] |
| Latency | input sampled once per logic step, not per event | [gen] |

**Choice D5 — touch scheme.** A: fixed on-screen pad + two buttons; B: virtual joystick (distance
selects walk / run); C: swipe and tap gestures.

| Criterion | W | A Fixed pad | B Joystick | C Gestures |
|---|---|---|---|---|
| Precision for walk / run / jump | 30 | 4 | 4 | 1 |
| Thumb ergonomics | 20 | 4 | 3 | 2 |
| Simplicity | 20 | 5 | 3 | 4 |
| Accessibility / remap | 15 | 4 | 3 | 2 |
| Reuse of retro-shmup touch code | 15 | 4 | 3 | 1 |
| **Total / 500** | 100 | **420** | 330 | 195 |

Points: A = 120+80+100+60+60; B = 120+60+60+45+45; C = 30+40+80+30+15. **Fallback:** B (330), a
different adapter behind `InputPort`, domain unchanged. Decision: fixed pad.

## 6. Enemies, blocks, items — detail

Rule of decision for every value below: **adopt the reference value** (ADR-0002, MCDA 485) unless
the table says "proposed change".

| Topic | Reference value | Tag | Consequence for the GDD |
|---|---|---|---|
| Walker speed | −8 = 0.5 px/frame (hard mode −12 = 0.75) | [src] *(disasm, `NormalXSpdData`)* | GDD §6 confirmed |
| Kicked shell speed | ±48 = 3 px/frame | [src] *(disasm, `KickedShellXSpdData`)* | GDD §6 confirmed |
| Demoted shelled walker | ±8 = 0.5 px/frame | [src] *(disasm, the demoted-walker speed table)* | note |
| Stunned shell revival | revive timers 16 and 11 interval units | [src] *(disasm, `RevivalRateData`)* | open: GDD has no revival (Q2) |
| Stomp bounce | vertical speed −4 for walkers and shelled walkers (shared `SBnce` routine); −3 only for a small group of classes (fish, projectiles, thrower classes) | [src] *(disasm, `EnemyStomped`, `SBnce`)* | GDD §4.2 (−4) confirmed |
| Death hop | vertical speed −4 | [src] *(disasm, `KillPlayer`)* | matches study §3 |
| Power-up emerging | rises 1 px every 4th frame until 16 px (≈ 64 frames), then moves at 16 = 1 px/frame | [src] *(disasm, `GrowThePowerUp`)* | GDD §8 confirmed |
| Star item | bounces with a jump routine (a jumping enemy's), moves like the growth item | [src] *(disasm, `PowerUpObjHandler`)* | later bricks |
| Floating score | a number rises for a fixed frame count (to measure) | [int] | Q3 |
| Flag pole | five height bands; the loop never compares the first table entry (24), so the effective cutoffs are y = 34, 80, 104, 144 px and everything above 34 gets the top band; scores 5000 / 2000 / 800 / 400 / 100 from the top | [src] thresholds *(disasm, `FlagpoleYPosData`)*; [int] scores | GDD §8 confirmed (five bands) |
| End-of-level sequence | hero slides down the pole, jumps off to the right, walks right automatically (forced input), enters the building drawn behind it (priority bit), then the time-to-points tally runs, then the next level loads | [src] *(disasm, `FlagpoleRoutine`, `PlayerEndLevel`, `StarFlagTaskControl`)* | add to GDD §9 flow |
| Bonus rule | coins collected in the third level of a world are compared with a per-world minimum; if enough, a hidden extra-life block is enabled in the **first level of the next world** (also enabled when starting a game) | [src] *(disasm, `Hidden1UpCoinAmts`)* | not in the slice; later bricks |
| Brick bump | a bumped block moves up 4 px then down (state machine with two block objects for brick chunks) | [src] *(disasm, `BlockObjectsCore`)* | brick-chunk effect is ours |
| Block bump placement | once a head hit is detected, the bumped block object's start y is offset by 4 px (big hero) or 18 px (small hero), chosen by hero size; not two collision probes | [src] *(disasm, `PlayerHeadCollision`, `BlockYPosAdderData`)* | collision brick (#5) |
| Hard-mode flag | a second difficulty set (faster walkers) selected by game progress | [src] *(disasm, `PrimaryHardMode`)* | not in 1.0 (YAGNI) |

## 7. Open questions for the GDD (to settle with the owner, GDD revision PR)

1. **Q1** Hit invulnerability: about 2.5 to 2.8 s as in the reference, or the GDD's 2 s?
2. **Q2** Do stunned shells revive (reference: yes, after about 3.9 to 5.6 s) or stay still (GDD §6)?
3. **Q3** Measure animation cadence, floating score duration and sprite sizes (a short follow-up
   pass, needs a frame-by-frame video reference; physics-like values are done).
4. **Q4** *(withdrawn: the reference stomp bounce of walkers is −4, same as the GDD; the −3 group is out of the slice)*.
5. **Q5** Level-time length per level (400 / 300 / 200) as in the reference?
6. **Q6** Contact test every frame (recommended) instead of every other frame.
7. **Q7** Pause key set and pause behaviour on tab blur (auto-pause), proposed yes.

## 8. Not covered here (other bricks)

Level grammar and layout rules (#4), collision details (#5), assets and sprite sheets (#6), UI and
catalogue (#7), save and meta (#8), QA and balance (#9, #10). Items needing a video reference are
listed in Q3.

## 9. Structure choice — record

Owner validated option A (single annex), MCDA 440 / 500. B (extend the main study) 335, C (six
annexes) 420. Fallback C if this file passes ~700 lines.

## Sources

- Main study sources (see [`reference-study.md`](reference-study.md) § Sources), in particular
  doppelganger, "SMBDIS.ASM — a comprehensive disassembly" (2012), mirror
  <https://gist.github.com/1wErt3r/4048722>. Routines and tables read for this annex:
  `DecTimers`, `RunGameTimer`, `GameTimerData`, `PauseRoutine`, `ScrollHandler`, `ScrollScreen`,
  `ChkPOffscr`, `GetScreenPosition`, `MusicHandler`, `LoadEventMusic`, `MusicLengthLookupTbl`,
  `Square1SfxHandler`, `PlayerEnemyCollision`, `EnemyStomped`, `StompedEnemyPtsData`,
  `NormalXSpdData`, `KickedShellXSpdData`, `RevivalRateData`, `PowerUpObjHandler`,
  `FlagpoleYPosData`, `PlayerEndLevel`, `BlockObjectsCore`, `ForceInjury`, `Hidden1UpCoinAmts`.
  Read for verification only; no code or data is copied.
- nesdev wiki — APU: <https://www.nesdev.org/wiki/APU>
- nesdev wiki — PPU palettes: <https://www.nesdev.org/wiki/PPU_palettes>
- nesdev wiki — Cycle reference chart: <https://www.nesdev.org/wiki/Cycle_reference_chart>
