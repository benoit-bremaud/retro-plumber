# Local review — retro-plumber (branch `docs/uml-specs`, brick 4, issue #6)

- Date: 2026-10-19
- Commit: 3539f76 + review fixes (see git log)
- Scope: branch diff main...HEAD (docs-only)
- Checks run: docs (doc-auditor) · design/correctness (consistency review vs GDD v1.0, ADR-0005/0006) · secrets
- Verdict: Must fixed, all Should applied; Must 2 (fixed) · Should 12 (fixed) · Nice 4 (2 fixed, 2 deferred)

## Pre-merge gate
| Step | Result |
|---|---|
| 1 Tests | not applicable (docs-only, no code) |
| 2 Coverage | not applicable (tests); docs: every diagram has source/context/notes blocks |
| 3 SonarQube | not applicable (no code, no scan configured yet) |
| 4 Secrets | ✅ gitleaks clean (7 commits) · no tracked .env · diff read: nothing sensitive |
| 5 QA pass | ✅ docs diff: all 9 Mermaid diagrams render with mmdc, 0 errors; no UI to run |
- Gate: **GREEN for a docs-only PR** (steps 1–3 not applicable)

## 🔴 Must Have (all fixed)
- `diagrams/meta/07,08` `[design]` · domain scenes called `InputPort` directly · routed through the app snapshot, app performs storage writes.
- `diagrams/gameplay/06` `[design]` · `Clock` called the app (adapter → app) · `FrameLoop` now pulls `now()` from its own `Clock` interface.

## 🟡 Should Have (all fixed)
- `use-case-specifications.md` `[docs]` · timer 400 retagged [gen]; 4b1 source fixed; scoring [gen] scope narrowed (OQ3); UC3 include covers whole UC4; UC1→UC4 relation added; UC2 level wording; Esc removed from pause trigger; extension order.
- `diagrams/system/01-use-case.md` `[docs]` · extend/include edges added.
- `diagrams/gameplay/04-class-domain.md` `[design]` · `LevelRun.paused`, `pauseLockFrames` added; `flagBandPoints(row)`.
- `diagrams/meta/08` `[docs]` · missing alternatives (reserved key, cancel, storage failure) drawn; `captureNext` removed.
- `diagrams/meta/07` `[docs]` · name charset marked as open point; `qualifies` call removed.
- `diagrams/system/02` `[docs]` · stray blank line; OQ2 pointer.

## 🔵 Nice to Have
- Fixed: `draw(state)` without `alpha`; ADR-0005 cited instead of CLAUDE.md only.
- Deferred: list `review-local.md` in the architecture README index; re-check status line after owner acceptance.

## ⚪ Disagree / Intentional
- UC4 and UC6 have no sequence (matrix G2, MCDA D2 A 400/500).

## Per-dimension summary
- Design: sequences now respect the dependency rule · Docs: complete, tags corrected · Tests: n/a · Security: no secrets, no personal data beyond a local name · Correctness: matrix cells reference existing elements; MCDA totals recompute.
