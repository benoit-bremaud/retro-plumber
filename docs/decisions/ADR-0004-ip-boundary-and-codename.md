# ADR-0004: IP boundary and the retro-plumber codename

**Status:** Accepted — owner, 2026-10-16.

## Context

The reference game, its characters and its title are trademarks and copyrighted works of their
publisher. The repository is public from creation (owner rule). Game rules and mechanics are not
protected by copyright; their expression (names, art, sound, music, level layouts, story) is.
The owner set that every game of the portfolio is named `retro-<word>`.

Options for the codename: **A** `retro-platformer`; **B** `retro-plumber`; **C** `retro-jumper`.

## MCDA (as presented to the owner)

| Criterion | Weight | A retro-platformer | B retro-plumber | C retro-jumper |
|---|---|---|---|---|
| Consistency with `retro-<genre>` siblings | 30 | 5 — genre word | 3 — a role, not a genre | 4 — action word |
| IP safety | 30 | 5 — fully generic | 3 — evokes the reference's hero | 5 — generic |
| Clarity | 20 | 4 | 4 | 4 |
| Availability (GitHub / npm, estimate, not checked) | 20 | 3 — common word | 4 | 4 |
| **Total / 500** | 100 | 440 | 340 | 430 |

## Decision

**`retro-plumber`**, chosen by the owner over the matrix's top score. The owner's choice stands; the
risk it carries is mitigated by the rules below.

**Fallback:** A, `retro-platformer` (440). Activation: rename the repository and folder only — a
drop-in, to use if the name ever draws an IP complaint.

- No name, logo, sprite, tile, sound, music, level layout or story of the reference anywhere in
  the repository (code, assets, docs, commits, issue titles). Source titles and URLs are allowed
  in citations only.
- The hero is our own character (GDD §3); "plumber" is never paired with the reference's visual
  identity (red cap, overalls, moustache).
- Disclaimer in the README: not affiliated with nor endorsed by any console publisher.

## Consequences

- The commercial title stays open; the codename may change before any release.
- Every asset brick checks its art against the reference's trade dress.
