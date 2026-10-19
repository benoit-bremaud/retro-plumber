# Traceability matrix — retro-plumber

**Version:** 0.2 — 2026-10-19 (brick 5, issue #7, adds sequences 10–14; v0.1 from brick 4, issue #6 was accepted). **Status:** Proposed (owner validation pending).

UC ↔ GDD ↔ sequence ↔ component ↔ class ↔ state ↔ data-flow. Use cases:
[`use-case-specifications.md`](use-case-specifications.md). Diagram numbers refer to the files in
[`diagrams/`](diagrams/). "—" means the diagram type has no element for that UC, with the reason
in the notes (never silently empty).

| UC | GDD | Sequence | Component | Class | State | Data-flow |
|---|---|---|---|---|---|---|
| UC1 Play a level | §4–§10 | 06 tick, 10 collisions, 11 item pickup, 12 flag and tally, 13 death and retry | 03 (FrameLoop, Game, Physics, Level, Entities, all ports) | 04 (Session, LevelRun, Hero, pools) | 02 flow, 05 hero | — (no persisted data) |
| UC2 Pause / resume | §8, §9 | 06 (pause branch), 14 pause and resume | 03 (InputPort, FrameLoop) | 04 (LevelRun) | 02 (Playing ⇄ Paused) | — |
| UC3 Name entry | §9 | 07 (13 hand-off) | 03 (Store, StoragePort; `save` not drawn) | G1 | 02 (NameEntry) | 09 |
| UC4 Hall of fame | §9 | G2 (13 hand-off) | 03 (as UC3) | G1 | 02 (HallOfFame) | 09 |
| UC5 Rebind controls | §9 | 08 | 03 (Keys, StoragePort; `bindings` not drawn) | G1 | G3 | 09 |
| UC6 Language / comfort | §9, §12 | G2 | 03 (`i18n`, `save` not drawn) | G1 | G3 | 09 |

## Gaps (recorded, not hidden)

| # | Gap | Decision |
|---|---|---|
| G1 | The class diagram covers gameplay only; `save`, `bindings`, `i18n`, scenes have no class. | Follow-up: meta class diagram at the first code brick; the modules appear in 03 notes. |
| G2 | UC4 and UC6 have no sequence (a single read / a single write, no multi-party interaction). | Chosen scope (MCDA D2: A 400/500, fallback B 340/500 adds them). |
| G3 | No state diagram for the options screen (OQ2 undecided). | Wait for the GDD answer to OQ2. |

## Reverse check (every diagram element serves a UC)

| Diagram | Serves |
|---|---|
| 02 flow | UC1, UC2, UC3, UC4 |
| 03 component | all |
| 04 class | UC1, UC2 |
| 05 hero | UC1 |
| 06 tick | UC1, UC2 |
| 07 name entry | UC3, UC4 |
| 08 rebind | UC5 |
| 09 persistence | UC3–UC6 |
| 10 collisions | UC1 |
| 11 item pickup | UC1 |
| 12 flag and tally | UC1 |
| 13 death and retry | UC1, UC3, UC4 (hand-off) |
| 14 pause and resume | UC2 |

## Decision record (MCDA)

Weights validated by the owner on 2026-10-19; figures are estimates [gen].

**D1 — where to keep the specs** (single file chosen).

| Criterion | Weight | A one file per UC | B one file | C in diagram notes |
|---|---|---|---|---|
| Traceability and links | 30 | 4 | 3 | 2 |
| KISS / low drift | 25 | 3 | 5 | 3 |
| Review size | 15 | 3 | 4 | 2 |
| Cockburn fidelity | 15 | 5 | 4 | 2 |
| Repository consistency | 15 | 4 | 4 | 2 |
| **Total / 500** | 100 | 375 | **395** | 225 |

Sensitivity: Traceability 40 / KISS 15 flips to A 385, B 375 (fragile result). **Fallback:** A,
mechanical split into six files.

**D2 — new diagrams** (A chosen).

| Criterion | Weight | A 3 sequences + 1 data-flow | B 6 sequences + 1 data-flow | C none |
|---|---|---|---|---|
| Matrix completeness | 30 | 4 | 5 | 1 |
| UML-first value | 25 | 4 | 4 | 1 |
| KISS / YAGNI | 25 | 4 | 2 | 5 |
| Review size | 20 | 4 | 2 | 5 |
| **Total / 500** | 100 | **400** | 340 | 280 |

Sensitivity: Completeness 45 / YAGNI 10 → A 400, B 385 (A holds). **Fallback:** B, adds the two
trivial sequences (UC4, UC6).

**D3 — sequences for the critical flows** (brick 5, issue #7; B chosen). Weights to be validated by
the owner; figures are estimates [gen].

| Criterion | Weight | A one per bullet (6) | B 5 diagrams, stomp + block hit grouped | C 3 diagrams, pause stays in 06 |
|---|---|---|---|---|
| Issue coverage | 30 | 5 | 5 | 3 |
| Cross-component value | 25 | 3 | 4 | 3 |
| KISS / YAGNI | 20 | 2 | 4 | 5 |
| Review size | 15 | 2 | 3 | 5 |
| Drift risk with 06 | 10 | 3 | 4 | 5 |
| **Total / 500** | 100 | 325 | **415** | 390 |

Sensitivity: KISS 35 / coverage 15 flips the order (B 400, C 420, A 280): the result is
sensitive to the coverage weight. **Fallback:** C, drop 11 and 14 (merge pickup into 10, pause stays in 06).

**D4 — participant granularity** (A chosen; brick 5, issue #7; weights to be validated by the owner, figures are estimates [gen]).

| Criterion | Weight | A domain modules of 04 | B one "Game" box |
|---|---|---|---|
| Traceability to 03 and 04 | 40 | 5 | 2 |
| Readability | 25 | 3 | 5 |
| UML-first design value | 35 | 5 | 2 |
| **Total / 500** | 100 | **450** | 275 |

Sensitivity: A stays first with readability 50 / traceability 25. **Fallback:** B.
