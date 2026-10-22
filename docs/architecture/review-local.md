# Local review — retro-plumber (branch `docs/uml-sequences`, brick 5, issue #7)

- Date: 2026-10-19
- Checks run: docs (doc-auditor) · design/correctness (consistency review vs GDD v1.0, ADR-0005/0006, diagrams 02–06) · secrets
- Verdict: Must 0 · Should 10 (all fixed) · Nice 9 (5 fixed, 4 deferred)

## Pre-merge gate
| Step | Result |
|---|---|
| 1 Tests | not applicable (docs-only, no code) |
| 2 Coverage | not applicable (tests); docs: 5 of 5 new diagrams carry source/context/notes blocks |
| 3 SonarQube | not applicable (no code, no scan configured yet) |
| 4 Secrets | ✅ gitleaks clean (9 commits, full history) · no tracked .env · diff read: nothing sensitive |
| 5 QA pass | ✅ all new Mermaid diagrams render with mmdc, 0 errors; no UI to run |
- Gate: **GREEN for a docs-only PR** (steps 1–3 not applicable)

## 🔴 Must Have
_None._

## 🟡 Should Have (all fixed)
- `diagrams/gameplay/14` `[design]` · one pause bit let a blur resume the game and a blur inside the 43-frame lock was ignored · snapshot now has a toggle bit and a pause-only auto-pause bit that bypasses the lock; lock re-armed on resume (tagged [gen]).
- `diagrams/gameplay/10` `[design][docs]` · `stompChain` credited to `Session` (lives on `Hero`, 04) · split into two calls; tag sentence deduplicated; links added.
- `diagrams/gameplay/11` `[design][docs]` · block coin shown as collected by contact · coin branch limited to level-placed coins, block coin credited at the hit.
- `traceability-matrix.md` `[docs]` · UC3/UC4 cells list the 13 hand-off; D4 gets owner-validation line and a complete sensitivity (weights sum to 100); reverse-check rows in numeric order.

## 🔵 Nice to Have
- Fixed: stomp-stops-shell note (10), `SmallInvulnerable` state name (13), "(slice)" wording (12), GDD §13 decision references (12, 13), link consistency.
- Deferred: `Contacts` dependencies on `Session`/`Hero`/pools missing in class diagram 04 (next 04 revision); Esc listed as a pause trigger in 02 versus 14 (tied to OQ2, GDD revision); tally scored via `Sess` directly in 12 (scene-owned, kept); legend of [src]/[int]/[gen] in README (GDD §2 is the source).

## ⚪ Disagree / Intentional
- Matrix v0.2 is Proposed until owner validation · 14 does not draw the Esc menu (OQ2) · 11/12 use `ItemPool`/`Contacts` from 04, absent from 03 (D4).

## Per-dimension summary
- Design: consistent with GDD/ADRs, domain never calls a port · Docs: complete blocks, links resolve · Tests: n/a · Security: no code, gitleaks clean · Correctness: figures and tags match the GDD, MCDA arithmetic verified.

---

# Local review — retro-plumber (branch `chore/sonarqube`, repository conformity)

- Date: 2026-10-22 · Commit: b4dd2c4 · Scope: `main...HEAD` (12 files, config and docs only, no code)
- Verdict: ship · Must 0 · Should 1 · Nice 1

## Pre-merge gate (head SHA b4dd2c4)
| Step | Result |
|---|---|
| 1 Tests | n/a, no code or suite yet (brick 0 design study) |
| 2 Coverage | n/a, no code (tests) · no doc counter applicable |
| 3 SonarQube | ✅ PASSED (project `retro-plumber`, 25 files, secrets and text sensors only) |
| 4 Secrets | ✅ gitleaks clean (`main..HEAD`) · no tracked .env · diff read: nothing sensitive |
| 5 QA pass | ✅ docs/config diff, no product to start · all 5 YAML files parse |
- Gate: **GREEN for push**. Workflows only run after the push, so their first run is verified on the PR.

## 🔴 Must Have
_None._

## 🟡 Should Have
- `SECURITY.md:8` `[security]` · private vulnerability reporting is **disabled** on the repository, yet the policy tells reporters to use it · owner enables it in Settings → Code security (a repository setting, not changed by the agent).

## 🔵 Nice to Have
- `.github/dependabot.yml` `[correctness]` · only the `github-actions` ecosystem and CodeQL `actions` are configured · add `npm` and `javascript-typescript` with the first code brick (noted in both files).

## ⚪ Disagree / Intentional
- No `ci.yml`, ESLint, tsconfig, Husky or Makefile: they belong to the first code brick (CLAUDE.md: no implementation before retro-shmup's vertical slice).
