# Local review — retro-plumber (branch docs/gdd-v1 vs main)

- Date: 2026-10-19
- Commit: 8bd662b
- Scope: branch diff main...HEAD (GDD v1.0, level-study rename, two diagram labels)
- Checks run: consistency/design · docs · security (secrets). Tests, coverage and SonarQube not applicable (docs-only, no code, no CI).
- Verdict: ship as-is · Must 0 · Should 2 (fixed) · Nice 9 (3 fixed, rest accepted)

## Pre-merge gate (head SHA 8bd662b)
| Step | Result |
|---|---|
| 1 Tests | n/a — docs-only repo |
| 2 Coverage | n/a (no code) |
| 3 SonarQube | not run — no project configured yet |
| 4 Secrets | gitleaks clean on main..HEAD · no tracked .env · diff read: nothing sensitive |
| 5 QA pass | n/a — no runnable product; Mermaid edits are label-only |
- Gate: **GREEN for push**; merge awaits owner validation.

## 🔴 Must Have
_None._

## 🟡 Should Have
- `game-design-document.md:66` `[docs]` · dangling cross-reference `(§13, A)` · fixed → `(§13, Q3)`.
- `game-design-document.md:3` `[docs]` · "no proposed item" overclaimed while ADR-0001/2/3/5 are still Proposed · fixed → scoped to GDD items.

## 🔵 Nice to Have
- Missing tags on shell wobble, "200 columns", flag-pole bracket wording, flag test wording · fixed.
- Stale `m`/`c` ids in level-study after rename · noted in its status line.
- Hero-state diagram lacks growth-item-during-invulnerability transition; game-flow diagram lacks auto-pause/Esc trigger · deferred to the UML brick.
- §11 table figures untagged (recalled from studies) · accepted.

## ⚪ Disagree / Intentional
- Old names remain only in the GDD rename record and PROJECT_LOG history (traceability).

## Per-dimension summary
- Design: matrices consistent (weights = 100, winner > runner-up = fallback) · Docs: cross-refs and tags fixed · Tests: n/a · Security: gitleaks clean · Correctness: geometry, counts and frame figures match the studies.
