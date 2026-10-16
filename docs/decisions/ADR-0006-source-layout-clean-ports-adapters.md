# ADR-0006: Source layout — Clean dependency rule, ports and adapters

**Status:** Accepted — owner, 2026-10-16.

## Context

Owner rule: Clean architecture everywhere and SOLID always, applied proportionally (no empty
layers). The domain must be testable without a browser (ADR-0005). The siblings use
`src/{app,adapters,domain}` with `domain/ports/`.

Options: **A** retro-shmup layout (`domain` with `ports`, `app`, `adapters`); **B** the four
full Clean rings (`entities`, `use-cases`, `interface-adapters`, `frameworks`); **C** flat
(`lib/` + `main.ts`).

## MCDA (as presented to the owner)

| Criterion | Weight | A retro-shmup layout | B Four rings | C Flat |
|---|---|---|---|---|
| Clean dependency rule and SOLID | 30 | 5 — domain pure, ports at the browser seam | 5 — same rule | 2 — logic and IO mixed |
| KISS / YAGNI (no empty layers) | 25 | 4 — three folders, all used | 2 — pass-through use-case layer for a single-player game | 5 |
| Fit with sibling modules | 25 | 5 — same layout | 2 — every module re-filed | 3 |
| Testability | 20 | 5 | 5 | 3 |
| **Total / 500** | 100 | **475** | 350 | 320 |

Correction: C was shown to the owner as 310; the correct sum is 320 (60 + 125 + 75 + 60). The
ranking is unchanged.

Sensitivity: with *KISS* doubled (50, others scaled down), A still leads.

**Fallback:** B (350) — activation only if a real use-case layer appears (e.g. several game
modes orchestrating the same rules): a design change, the domain is untouched.

## Decision

```
src/domain/   pure rules (physics, level, entities, game, scenes, random, save, i18n, audio) and ports/
src/adapters/ browser implementations of the ports
src/app/      composition root and frame loop
```

Dependencies point `app → adapters → domain`. The domain is compiled with a `tsconfig.domain.json`
without the DOM lib, and an ESLint import boundary forbids `domain → adapters|app`.

## Consequences

- A port exists only at a real seam (browser API or test substitution).
- New enemies, tiles and items extend discriminated unions / behaviour tables (open/closed).
