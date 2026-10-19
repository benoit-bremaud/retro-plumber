# Local review — retro-plumber (branch docs/level-study, brick 2, issue #4)

- Date: 2026-10-19
- Commit: bc05fd2
- Scope: branch diff main...HEAD (one new file, `docs/study/level-study.md`)
- Checks run: independent reviewer (arithmetic, grid/text consistency, IP, tags) · gitleaks · IP-name grep
- Verdict: ship — Must 0 · Should 6 (all fixed) · Nice 8 (5 fixed, 3 noted)

## Pre-merge gate (head SHA bc05fd2)
| Step | Result |
|---|---|
| 1 Tests | not applicable (docs-only, no code yet) |
| 2 Coverage | not applicable |
| 3 SonarQube | not run (no SonarQube project for this repo yet) |
| 4 Secrets | gitleaks clean on the file · no tracked `.env*` · IP-name grep clean · diff read |
| 5 QA pass | not applicable (no product to run) |
- Gate: **GREEN for a docs-only diff**; steps 1–3 and 5 are not applicable or not available yet.

## Must Have
_None._ MCDA totals, jump arithmetic, grid/text columns and GDD §10 quotas were recomputed and match.

## Should Have (all fixed)
- Pit 3 was called "run-only" although a walking jump reaches ≈ 80 px → reworded "run intended", added Q13.
- E7 and palette rows tagged `[src]`/`[gen]` against GDD → retagged `[gen]`, GDD cited as source of the list.
- D5 sensitivity threshold (10 → 7) corrected.
- E2 distances: one convention (free columns).
- Wall limit rounding (70 % of apex) stated.

## Nice to Have
- Fixed: density wording, E1 reference point, "pole col" typo, D1 scores added, camera conflict raised as Q14.
- Noted: the 600-line fallback trigger is far from reached (337 lines).

## Disagree / Intentional
- "doppelganger's public disassembly" is a source citation, not an IP reference (same as brick 1).

## Per-dimension summary
- Design: rules E1–E7 are checkable predicates · Docs: tags consistent after fixes · Tests: layout test is a later brick (#9, #10) · Security: no secrets · Correctness: numbers recomputed
