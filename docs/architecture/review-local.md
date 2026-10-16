# Local review — retro-plumber (branch docs/study vs main, brick 0)

- Date: 2026-10-16
- Commit: 0d9fa4a + review fixes
- Scope: branch diff main...HEAD plus uncommitted fixes (documentation only, no code)
- Checks run: design (architecture-guardian) · docs (doc-auditor). Tests, security-code and
  correctness: not applicable, no code in this brick.
- Verdict: fix Must Have first — all Must resolved in this change · Must 6 (resolved) · Should 6 · Nice 3

## Pre-merge gate (head SHA: see commit)
| Step | Result |
|---|---|
| 1 Tests | N/A — no code or test suite exists yet (docs-only brick) |
| 2 Coverage | N/A — tests and doc coverage have nothing to measure |
| 3 SonarQube | N/A — nothing to scan; wired in the first code brick |
| 4 Secrets | gitleaks clean · no tracked .env · diff read: nothing sensitive |
| 5 QA pass | Docs-only: all 5 Mermaid diagrams render with mmdc, no running product exists |
- Gate: **GREEN for a docs-only merge** once the owner accepts steps 1–3 as not applicable.

## 🔴 Must Have (resolved)
- `docs/architecture/diagrams/gameplay/04-class-domain.md` `[design]` · missing Session, units,
  item/block model · rewritten.
- `docs/architecture/diagrams/gameplay/05-state-hero.md` `[design]` · Dead/FlagSlide/AutoWalk at
  wrong level · fixed.
- `docs/architecture/diagrams/system/03-component.md` `[design]` · Clock ownership, loop wiring ·
  fixed.
- `docs/design/game-design-document.md` `[docs]` · storage units undefined · units table added.
- `docs/decisions/ADR-0003-*.md` `[docs]` · option label D vs C · fixed.
- `CLAUDE.md` `[design]` · RNG wording contradicted GDD, `random` module kept · fixed, folder removed.

## 🟡 Should Have
- ADR-0002/0004 sensitivity checks · added.
- ADR-0005/0006 wording and arithmetic · fixed (ADR-0006 correction note).
- `PROJECT_LOG.md` duplicate "pre-push review" wording · fixed.
- Reference study wording kept generic (IP rule) · fixed.
- ADR statuses: 0001/0002/0003/0005 remain Proposed · owner to accept.
- GDD "(proposed)" items (cast names, hitboxes, scoring values) · owner to validate.

## 🔵 Nice to Have
- SECURITY.md, CONTRIBUTING.md, security CI (Gitleaks, CodeQL) · with GitHub repo creation.
- Cross-project shared-package issue on retro-shmup · file once the repo exists.
- Traceability matrix · after the first feature diagrams.

## ⚪ Disagree / Intentional
- Initial empty commit on `main` · mirrors retro-maze.

## Per-dimension summary
- Design: dependency rule consistent across CLAUDE.md, ADR-0006 and the component diagram · Docs:
  units and decisions now consistent · Tests: n/a · Security: no secrets · Correctness: physics
  values re-checked against the public disassembly.
