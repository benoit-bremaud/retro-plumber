# Level study and vertical-slice level plan

Status: **Accepted v0.2** (owner, 2026-10-19; brick 2, issue #4, epic #11). Companion of
[`reference-study.md`](reference-study.md) (v0.2, hero physics) and
[`reference-study-complement.md`](reference-study-complement.md) (v0.3, Accepted). Input for the GDD
(§7 levels, §10 vertical slice), not a design contract; open questions go to the later GDD
revision PR.

Same IP rule and tag legend as the earlier studies: generic role names and our own cast names
only (GDD §3); **[src]** read in a cited public source (here: the two earlier studies, which cite
the public documentation and the public disassembly), **[int]** our derivation from a sourced
value, **[gen]** genre knowledge or our own proposal, not checked against a source. The level
below is **entirely our own layout**: no third-party level, map, name or asset was used, and no
level of the reference game was consulted for its layout.

Jump ranges include the 4 px/frame terminal fall speed (GDD §4.2), computed in continuous time; the fixed-point update order may shift them by a few pixels (golden-trajectory test, ADR-0005).

Honest limit: the reference level grammar of the earlier study (§5 there) is [gen]. This annex
therefore takes its hard numbers from our own physics tables (derived, [int]) and treats the
rhythm and difficulty rules as design choices justified by MCDA, not as measurements of the
reference.

## 1. Inputs and derived limits

Every dimension of the level is bounded by what the hero can do. Values below are derived from
the GDD §4 tables (which mirror the main study §2–§3 [src]); flat-ground jump, take-off and
landing at the same height, hitbox width ignored.

| Quantity | Value | Tag |
|---|---|---|
| Screen | 256 × 240 px, 16 × 16 px tiles, 16 columns visible, 13 level rows | [src] GDD §3.1, main study §1 |
| Camera | one-way; scroll starts when the hero is at x ≥ 80 px; the right end of the level locks the scroll | [src] complement §2 |
| Walk speed (cap) | 1.5 px/frame (24 units) | [src] main study §2 |
| Run speed (cap) | 2.5 px/frame (40 units); reached after ≈ 45 frames, ≈ 3.5 tiles of run-up from rest (`228/4096` px/frame²) | [src] cap; [int] run-up |
| Jump, take-off speed 16–24 (walking): rise | v₀ = 4 px/frame, gravity held 30/256 → apex 68 px (4.3 tiles) after 34 frames; fall 96/256 reaches the 4 px/frame terminal speed (GDD §4.2) after 11 frames, then 4 px/frame → 22 frames | [int] from GDD §4.2 |
| Jump, walking: range | 56 frames × 1.5 px/frame ≈ **85 px = 5.3 tiles** (80 px = 5.0 tiles if the terminal clamp is ignored) | [int] |
| Jump, running (≥ 25): rise and range | v₀ = 5 px/frame, gravity 40/256 → apex 80 px (5 tiles); the fall of 80 px takes 24 frames with the clamp, 56 frames in all × 2.5 ≈ **139 px = 8.7 tiles** (122 px without the clamp) | [int] |
| Jump, standing (< 16 units): range | at most 0.94 px/frame × 53 frames ≈ 49 px (3.1 tiles) | [int] |
| Safe-design margin | a mandatory obstacle must need at most **75 %** of the capability of the speed it requires (pit width, wall height), so a late button press still succeeds | [gen] |
| Resulting limits | pit crossable at walking speed ≤ 4.0 tiles (64 px); run-intended pit ≤ 6.5 tiles (104 px); wall or pipe ≤ 3 tiles (48 px, 70 % of the 68 px apex, rounded down from 51 px) | [int] from the margin |
| Head-bump reach | a block whose underside is 3 tiles above the standing row can be bumped by any jump (the hero needs 48 − 14 = 34 px of rise when Small, 48 − 28 = 20 px when Big) | [int] |
| Level traversal time | start to flag ≈ 3040 px: 51 timer units at run speed, 84 at walk speed (1 unit = 24 frames); the GDD timer of 400 units is ≈ 4.7× the walking time (400 / 84.4) | [int] complement §1 |

Consequences: pit widths of 2, 3 and 5 tiles; pipes of 2 and 3 tiles; the 5-tile pit is the only
obstacle designed for a run (a walking crossing is not excluded, see Q13). The wide-pit run-up needs at least 4 free tiles before the edge.

## 2. Tile palette (from GDD §7, glyphs are ours)

| Glyph | Tile | Behaviour | Tag |
|---|---|---|---|
| `.` | empty | — | [gen] |
| `G` | ground | solid, 2 rows thick at the bottom (rows 11–12) | [gen] |
| `S` | hard block | solid, never breaks; stairs and pole base | [gen]; tile list from GDD §7 |
| `B` | brick | bumps when Small, breaks when Big | [gen]; tile list from GDD §7 |
| `?` | item block (coin) | gives a coin when bumped | [gen]; tile list from GDD §7 |
| `U` | item block (growth item) | looks like `?` in game; the glyph only marks the content in the data | [gen]; tile list from GDD §7 |
| `H` | hidden coin block | invisible until bumped from below | [gen]; tile list from GDD §7 |
| `[` `]` / `{` `}` | pipe top / body, 2 columns wide | solid, not enterable in the slice | [gen]; tile list from GDD §7 |
| `F` | flag pole | grab bands from top to bottom (five bands, GDD §8) | [gen]; five bands [src] complement §6 |
| `Z` | goal house | the hero walks into it after the pole | [gen]; tile list from GDD §7 |

Entities (hero start, Grubbit `m`, Dentshell `c`) are not in the grid. They are a separate list
(column, row, type), as GDD §7 specifies for the level format.

## 3. Level grammar

### 3.1 Sections and rhythm

The level is cut into sections of 16 to 40 columns, one to two screens each. Each section has one
purpose, and tension alternates with release.

| Section | Columns | Purpose | Intensity (0–5, ours) |
|---|---|---|---|
| S1 Warm-up | 0–31 | safe ground, first item blocks, first enemy alone | 1 |
| S2 Teach | 32–71 | pipes, brick run, enemy pairs, first small pit | 2 |
| S3 Test | 72–111 | 3-tile pit, shelled walker introduced alone, growth item | 3 |
| S4 Twist | 112–151 | breather, pyramid staircase, high brick route, secret block, mixed enemies | 3 |
| S5 Peak | 152–181 | tall pipe, run-intended pit, two item runs, last enemies | 5 |
| S6 Release | 182–199 | no enemy, climb the 8-step staircase, pole, house; time tally runs | 1 |

**Choice D2 — length and section count.** A: ≈ 200 columns in 6 sections (GDD §10 says ≈ 200);
B: 160 columns in 5 sections; C: 240 columns in 7 sections.

| Criterion | W | A 200 / 6 | B 160 / 5 | C 240 / 7 |
|---|---|---|---|---|
| Fidelity to GDD §7 and §10 | 30 | 5 — as written | 3 — drops ≈ 20 % | 3 — adds ≈ 20 % |
| Quota density (12 + 3 enemies, 3 pits, 6 brick runs read well) | 20 | 4 — one hazard (15 enemies + 3 pits) per ≈ 11 columns | 4 — denser, tighter | 3 — sparse stretches |
| Test cost (scripted run length) | 15 | 4 | 5 | 3 |
| Production cost (layout, art, review) | 15 | 4 | 5 | 3 |
| Room to teach each role alone (E4) | 20 | 5 | 3 | 5 |
| **Total / 500** | 100 | **450** | 380 | 340 |

Points: A = 150+80+60+60+100; B = 90+80+75+75+60; C = 90+60+45+45+100. Sensitivity:
A − B = 2·W_fidelity + 2·W_teach − W_test − W_prod (density has the same score), so B overtakes A
only if the two cost criteria together weigh more than twice fidelity plus teaching (now 30 against
100). Estimates: all scores are design judgments, not measurements. **Fallback:** B (380) — a
re-layout of the gaps with the quotas unchanged, not a drop-in. Decision: A.

### 3.2 Difficulty curve

**Choice D3 — curve shape.** A: teach, test, twist per role, with a release at the end (sawtooth
that rises overall); B: flat; C: monotonic rise to the flag.

| Criterion | W | A Sawtooth | B Flat | C Monotonic |
|---|---|---|---|---|
| Onboarding (each role met alone first) | 30 | 5 | 2 — no ramp | 3 |
| Genre rhythm (tension and release) | 25 | 5 | 2 | 3 |
| Measurable per section in a scripted run | 15 | 4 | 5 | 4 |
| Fairness and recovery after a mistake | 20 | 5 — breathers after each peak | 3 | 3 — no relief |
| Simplicity (KISS) | 10 | 3 | 5 | 5 |
| **Total / 500** | 100 | **465** | 295 | 335 |

Points: A = 150+125+60+100+30; B = 60+50+75+60+50; C = 90+75+60+60+50. Sensitivity: raising KISS
to 30 (taking 20 from onboarding) gives A 425, C 375, so A still leads. **Fallback:** C (335) —
remove the breathers; a drop-in change to the layout. Decision: A. The intensity column of §3.1 is
the target: 1, 2, 3, 3, 5, 1.

### 3.3 Enemy placement rules

The rules are written as checkable predicates, so a layout test (brick 9) can verify any level
against them.

| Rule | Statement | Tag |
|---|---|---|
| E1 | The first enemy is at least 24 columns (1.5 screens) from the hero's start column | [gen] |
| E2 | No enemy within 4 columns before a pit's take-off edge or after its landing edge | [gen], 4 = run-up [int] §1 |
| E3 | No enemy on a staircase and none in the last 14 columns | [gen] |
| E4 | A new enemy role first appears alone on flat ground, with at least 8 columns clear around it | [gen] |
| E5 | Pairs of the same role (2 to 4 columns apart) appear only after that role was met alone | [gen] |
| E6 | At most 4 enemies in any window of 16 columns (one screen) | [gen] |
| E7 | Activation and removal as in GDD §6: spawn column enters the screen + 1; removed 2 columns behind the camera | [gen] GDD §6, marked *(ours)* there |

**Choice D4 — placement method.** A: the rule set above, checked by a test; B: free placement by
feel; C: seeded procedural placement.

| Criterion | W | A Rules | B Free | C Seeded |
|---|---|---|---|---|
| Fairness and determinism | 25 | 5 | 3 | 2 — a bad seed is unfair |
| Testability (automatic check) | 25 | 5 | 1 | 3 |
| Authoring effort (KISS) | 15 | 3 | 5 | 2 |
| Feel fidelity (hand-tuned set pieces) | 20 | 4 | 4 | 2 |
| Reuse for later levels | 15 | 5 | 2 | 4 |
| **Total / 500** | 100 | **450** | 285 | 255 |

Points: A = 125+125+45+80+75; B = 75+25+75+80+30; C = 50+75+30+40+60. Sensitivity: effort at 40
and testability at 0 gives A 400, B 385, so A still leads, narrowly. **Fallback:** B (285) —
placement by feel with a manual review; drop-in (the rules become advice). Decision: A.

### 3.4 Secrets and optional routes

**Choice D5 — secrets in the slice.** A: one hidden coin block plus one optional high brick route;
B: none; C: a bonus zone behind an enterable pipe.

| Criterion | W | A Block + route | B None | C Bonus zone |
|---|---|---|---|---|
| Compliance with GDD §7 (hidden block listed; pipes not enterable) | 30 | 5 | 4 — palette tile unused | 1 — needs a new rule |
| Exploration reward | 20 | 3 | 1 | 5 |
| Cost (art, test) | 20 | 4 | 5 | 1 — a second room |
| Risk of new rules / determinism | 20 | 5 | 5 | 2 |
| Simplicity (KISS) | 10 | 4 | 5 | 1 |
| **Total / 500** | 100 | **430** | 390 | 200 |

Points: A = 150+60+80+100+40; B = 120+20+100+100+50; C = 30+100+20+40+10. Sensitivity: if the
reward weight fell to 0 and moved to cost, B (470) would overtake A (450). The reward weight
would have to fall below about 7 for that (A − B = 3·W_reward − 20 when the freed weight goes to cost). **Fallback:** B (390) — delete one block and one
brick line; drop-in. Decision: A.

## 4. Layout plan (original)

Grid: 200 columns × 13 rows, row 0 at the top, ground in rows 11–12. The ruler digit marks the
first column of each group of ten (block 100–149 starts at column 100). The hero starts at
column 2, row 10.

```text
cols 0-49
    0         1         2         3         4
r00 ..................................................
r01 ..................................................
r02 ..................................................
r03 ..................................................
r04 ..................................................
r05 ..................................................
r06 ..................................................
r07 ............?.....BUB?B.....................BBB...
r08 ..................................................
r09 ......................................[]..........
r10 ......................................{}..........
r11 GGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGG
r12 GGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGG

cols 50-99
    5         6         7         8         9
r00 ..................................................
r01 ..................................................
r02 ..................................................
r03 ..................................................
r04 ..................................................
r05 ..................................................
r06 ..................................................
r07 ..................................B?BUB...........
r08 ......[]..........................................
r09 ......{}........................................[]
r10 ......{}........................................{}
r11 GGGGGGGGGGGGGGGG..GGGGGGGG...GGGGGGGGGGGGGGGGGGGGG
r12 GGGGGGGGGGGGGGGG..GGGGGGGG...GGGGGGGGGGGGGGGGGGGGG

cols 100-149
    0         1         2         3         4
r00 ..................................................
r01 ..................................................
r02 ..................................................
r03 ..................................................
r04 ....................................B?BBB.........
r05 ..................................................
r06 ..................................................
r07 .....................................SS...........
r08 ....................................SSSS..........
r09 ...................................SSSSSS.........
r10 ..................................SSSSSSSS........
r11 GGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGG
r12 GGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGG

cols 150-199
    5         6         7         8         9
r00 ..................................................
r01 ..................................................
r02 ..........................................F.......
r03 ........................................S.F.......
r04 .......................................SS.F.......
r05 ......................................SSS.F.......
r06 .....................................SSSS.F.......
r07 H...................B?B...BB?BB.....SSSSS.F...ZZZ.
r08 ....[].............................SSSSSS.F...ZZZ.
r09 ....{}............................SSSSSSS.F...ZZZ.
r10 ....{}...........................SSSSSSSS.S...ZZZ.
r11 GGGGGGGGGGGG.....GGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGG
r12 GGGGGGGGGGGG.....GGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGG
```

### 4.1 Features by section

| Section | Features (column, row) |
|---|---|
| S1 | coin block (12, 7); brick run R1 cols 18–22 row 7: `B U B ? B` — growth block at 19, coin block at 21 |
| S2 | pipe P1 cols 38–39, 2 tiles high; brick run R2 cols 44–46 row 7; pipe P2 cols 56–57, 3 tiles high; pit 1 cols 66–67 (2 tiles) |
| S3 | pit 2 cols 76–78 (3 tiles); brick run R3 cols 84–88 row 7: `B ? B U B` — coin block 85, growth block 87; pipe P3 cols 98–99, 2 tiles high |
| S4 | staircase ST1 (pyramid) cols 134–141: up 1–4, flat top at cols 137–138, down 3–1; high brick run R4 cols 136–140 row 4: `B ? B B B` — coin block 137; hidden coin block H (150, 7) |
| S5 | pipe P4 cols 154–155, 3 tiles high; 6 flat columns 156–161 for the run-up; pit 3 cols 162–166 (5 tiles, run intended); brick run R5 cols 170–172 row 7: `B ? B` — coin block 171; brick run R6 cols 176–180 row 7: `B B ? B B` — coin block 178 |
| S6 | staircase ST2 cols 183–190, steps 1–8; pole col 192 (rows 2–9, base hard block row 10); goal house cols 196–198, rows 7–10 |

The camera cannot pass column 184 on the left edge (scroll lock at the level end, complement §2),
so the pole (screen x = 128 px) and the house (192–239 px) are both on screen. [int]

### 4.2 Entity list

All enemies stand on the ground, row 10. The hero starts at column 2.

| Id | Role | Column | Section | Note |
|---|---|---|---|---|
| m1 | Grubbit | 26 | S1 | first enemy, alone (E1, E4) |
| m2, m3 | Grubbit | 48, 50 | S2 | pair beside R2 (E5) |
| m4 | Grubbit | 60 | S2 | 5 free columns before pit 1 (E2) |
| m5 | Grubbit | 83 | S3 | 4 columns after pit 2's landing (E2) |
| c1 | Dentshell | 92 | S3 | first shelled walker, alone, 9 columns clear (E4) |
| m6, m7 | Grubbit | 106, 108 | S3 | pair |
| m8, m9 | Grubbit | 116, 120 | S4 | after a 4-column breather |
| c2 | Dentshell | 126 | S4 | 8 columns before the staircase (E3) |
| m10, m11 | Grubbit | 146, 148 | S4 | pair after the staircase |
| m12 | Grubbit | 175 | S5 | between R5 and R6 |
| c3 | Dentshell | 178 | S5 | under R6 |

### 4.3 Quota and rule check (GDD §10)

| Item | GDD §10 | Plan | Counted as |
|---|---|---|---|
| Pits | 3 | 3 | pits 1, 2, 3 |
| Brick rows | 6 | 6 | runs R1–R6 (interpretation: one brick row per run) |
| Item blocks | 8 (6 coin, 2 growth) | 8 (6 coin, 2 growth) | coin: 12, 21, 85, 137, 171, 178; growth: 19, 87 |
| Pipes | 4 | 4 | P1–P4 |
| Staircases | 2 | 2 | ST1 (pyramid), ST2 (final) |
| Grubbits | 12 | 12 | m1–m12 |
| Dentshells | 3 | 3 | c1–c3 |
| Flag pole, goal house | 1 each | 1 each | cols 192 and 196–198 |
| Hidden coin block | not in §10 | 1 | extra (D5), not counted in the 8 |

| Check | Result |
|---|---|
| Walking-speed pits ≤ 4.0 tiles | pit 1 = 2, pit 2 = 3 ✓ |
| Run-intended pit ≤ 6.5 tiles, above the 4.0 walking limit | pit 3 = 5 ✓; a walking jump reaches ≈ 85 px (5.3 tiles) with the hitbox ignored, so a walking crossing is possible but not safe: the run is intended, not enforced (see Q13) |
| Pipes ≤ 3 tiles | 2, 3, 2, 3 ✓ |
| Run-up before pit 3 ≥ 4 free tiles | 6 (cols 156–161) ✓ |
| E1 first enemy ≥ 24 columns from the start (column 2) | 26 − 2 = 24 ✓ |
| E2 free columns around pits | pit 1: 5 before, 15 after; pit 2: 15 before, 4 after; pit 3: 13 before, 8 after (free columns between the pit edge and the enemy) ✓ |
| E3 no enemy on a staircase or after column 185 | last enemy at 178 ✓ |
| E4 new role alone, ≥ 8 columns clear | m1 at 26 (nearest 22 away), c1 at 92 (9 and 14 away) ✓ |
| E6 ≤ 4 enemies in 16 columns | maximum found = 4 (columns 106–121) ✓ |
| Staircase top to pole | 190 → 192, 1 empty column ✓ |

Checked with a throwaway script over the grid and the list (not committed); a committed layout
test is a later brick (#9, #10).

### 4.4 Timing

Start to flag: 190 columns = 3040 px → 51 timer units at run speed, 84 at walk speed. With the
400-unit timer there are over 300 units of margin; this is input to open question Q5 of the
complement study (400 / 300 / 200 per level).

## 5. Open questions for the GDD revision

1. **Q8** Flag grab bands: the reference cutoffs (y = 34, 80, 104, 144 px, complement §6) are in
   its screen coordinates; the mapping to our pole (rows 2–9, 8 tiles = 128 px) must be fixed in
   the collision brick (#5) and the GDD.
2. **Q9** Confirm the interpretation "6 brick rows = 6 brick runs" and "2 staircases = one
   pyramid + one final climb" of GDD §10.
3. **Q10** The hidden coin block is outside the GDD §10 count (8 item blocks); add it to §10 or
   keep it as an unlisted extra.
4. **Q11** Level timer for the slice (links Q5): the traversal times above suggest even 200 units
   is generous.
6. **Q13** Pit 3 (5 tiles, 80 px) is within the walking range (≈ 85 px) once the 12 px hitbox
   is counted: either accept it as "run intended", or widen it by a tile and relax the 75 %
   margin for it. A golden-trajectory test (ADR-0005) can settle it.
7. **Q14** GDD §7 says the camera follows once the hero "passes the screen centre"; the
   complement §2 says x ≥ 80 px. The GDD should be aligned in the revision PR.

## 6. Structure choice — record

Owner validated option A (single annex file), MCDA 445 / 500, 2026-10-19. B (two files) 395,
C (layout in the GDD) 270. Weights: traceability 25, KISS 20, consistency with the brick 1 annex
20, reviewability 20, reuse by later bricks 15. Scores in the weight order above — A: 5, 5, 5, 3, 4; B: 4, 3, 3, 5, 5; C: 2, 4, 1, 3, 4. Fallback B if this file passes about 600 lines.

## Sources

- Hero physics, camera and flag-band values: [`reference-study.md`](reference-study.md) and
  [`reference-study-complement.md`](reference-study-complement.md) (public community documentation
  and doppelganger's public disassembly, read for verification only; nothing copied).
- Screen, jump, gravity and scoring values: [`../design/game-design-document.md`](../design/game-design-document.md)
  §3.1, §4, §6, §7, §8, §10.
- Level layout, rhythm rules and difficulty curve: our own design ([gen]); no third-party level
  was consulted.
