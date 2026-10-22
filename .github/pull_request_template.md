<!--
  Title format (Conventional Commits): <type>(<scope>): <short description>
  Types:  feat, fix, refactor, docs, test, chore, perf, style, build, ci
  Scopes: design, study, adr, uml, docs, ci, repo (code scopes once code exists)
-->

## Summary

<!-- One short paragraph or 2-4 bullets. What this PR does and why. -->

-

## Context

<!-- Optional: GDD sections, ADRs, UML diagrams, related PRs, trade-offs to notice. -->

## Checklist

- [ ] Conforms to the GDD and the relevant ADRs (`docs/decisions/`); no validated decision reopened silently
- [ ] UML diagrams validated under `docs/architecture/diagrams/<feature>/` for any new feature
- [ ] No third-party intellectual property (names, sprites, sounds, layouts) anywhere
- [ ] Figures tagged `[src]` / `[int]` / `[gen]` where the GDD requires it
- [ ] No secret in the diff (gitleaks clean)
- [ ] Code PRs only: tests cover happy path, sad path and edge cases; no `any`, no default exports; the domain imports no browser API
- [ ] Pre-push review gate (`review-local`) green
- [ ] `PROJECT_LOG.md` will be updated after merge

## Linked issue

Closes #
