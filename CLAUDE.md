# CLAUDE.md — retro-plumber

## Project overview

**Name:** retro-plumber (codename, owner choice 2026-10-16 — the commercial title is still open).
**Type:** side-scrolling platformer for the browser, in the tradition of the 1985 home-console
genre, with original character, levels, art and sound. Planned stack: TypeScript strict + native
Canvas 2D + Vite, same as `../retro-shmup` (see ADR-0001, proposed).
**Scope:** a faithful-feel platformer; a first vertical slice of one overworld level, then
extension level by level.
**Status:** **brick 0 — design study** (GDD, ADRs, UML study). No game code until
`../retro-shmup` has shipped its vertical slice (owner decision 2026-10-16).
**Language:** English only in the repository (code, comments, commits, docs, issues, PRs).

Global rules (Conventional Commits, branch from `main`, no `any`, no default exports, pre-push
review gate, MCDA for every choice, security defaults) are inherited from `~/.claude/CLAUDE.md`
and `~/.agent-rules/common.md` — not duplicated here. Below are the rules that apply **only** to
this repository.

## Architecture — Clean everywhere, SOLID always (owner rule 2026-10-16)

**The dependency rule never bends:** source dependencies point inward,
`app → {adapters, domain}`, `adapters → domain`, `domain → ∅`; never the reverse.

- `src/domain/` is pure TypeScript: no `window`, `document`, `canvas`, `AudioContext`,
  `localStorage`, `performance`, `Date.now()` or `Math.random()`. Enforced by a dedicated
  `tsconfig.domain.json` (no DOM lib) and an ESLint import boundary, as in retro-shmup.
- `src/domain/ports/` declares the interfaces the domain needs (`RenderPort`, `InputPort`,
  `AudioPort`, `StoragePort`; the frame loop in `src/app/` owns its own small `Clock` interface). Each port is small and client-specific (ISP).
- `src/adapters/` implements the ports with browser APIs; adapters depend on the domain, never
  on each other.
- `src/app/` is the composition root: it builds the adapters, injects them (DIP) and runs the
  frame loop. It is the only place that knows every concrete class.

**SOLID, applied as principles, not ceremony:**

- **S** — one reason to change per module: physics, collision, level data, entity behaviour,
  scoring and rendering live in separate modules.
- **O** — new enemies, power-ups and tile kinds are added as new entries of a discriminated
  union / behaviour table, not by growing an `if/else` in the update loop.
- **L** — every port implementation (real or test fake) honours the same contract.
- **I** — no "god" port: the renderer never sees input, the audio never sees storage.
- **D** — the domain depends on ports; concrete adapters are injected by `src/app/`.

**Proportionality (KISS / YAGNI):** a port exists only where a real seam exists (browser API or
test substitution). No empty pass-through layers, no abstraction "for later". Generic modules
are reused from retro-shmup; copy or shared package is decided at the first code brick
(ADR-0003).

## Non-negotiables (project)

- **No third-party intellectual property.** No original-game name, logo, sprite, sound, music,
  level layout or character name — in code, assets, docs, commit messages or issue titles.
  Rules and physics are described from public behavioural references; everything shown or heard
  is our own.
- **Design before code.** A feature is implemented only after (1) its rules exist in the GDD and
  (2) its UML diagrams are validated. Diagrams are Mermaid under
  `docs/architecture/diagrams/<feature>/`.
- **Validated decisions are not reopened silently.** Changing an ADR or a GDD rule requires a
  superseding ADR or a GDD revision in its own PR.
- **Fixed 60 Hz timestep, deterministic simulation.** Accumulator loop, sub-pixel fixed-point
  physics; any future random rule draws from a seeded RNG. The same input script must replay
  identically.
- **No allocation inside the game loop.** Preallocated entity pools, index loops only.
- **Every player-facing string goes through the message catalogue** (EN default, FR).
- **Every game-feel effect is toggleable** and respects photosensitivity options.
- **Tests:** domain unit tests, headless level-simulation tests with scripted inputs and a fixed
  one browser smoke test. Render and audio are mocked at their ports, never the domain.

## Repository layout

```
docs/design/        GDD and design annexes
docs/study/         behavioural study of the reference genre (physics, enemies, level grammar)
docs/decisions/     ADRs (Nygard format, ADR-NNNN-<slug>.md)
docs/architecture/  UML diagrams (Mermaid) and QA evidence (qa/)
public/assets/      original sprites and audio
src/domain/         pure rules and ports: physics, level, entities, game, scenes, save,
                    i18n, audio                                          — later bricks
src/adapters/       browser implementations of the ports                — later bricks
src/app/            composition root and frame loop                     — later bricks
tests/              Vitest, mirroring src/, plus smoke/ and support/    — later bricks
```

## Commit scopes (Conventional Commits)

`design` (GDD), `study`, `adr`, `uml`, `docs`, `ci`, `repo`, and — once code exists — `engine`,
`physics`, `level`, `entities`, `render`, `audio`, `input`, `ui`, `assets`, `i18n`, `tests`.

Branch naming: `feat/<scope>`, `fix/<scope>`, `refactor/<scope>`, `chore/<scope>`, `docs/<scope>`.

## Workflow reminders

- Open a branch for every change; `main` is protected once the GitHub repository exists.
- Run the local pre-push review gate (`review-local`) before every push.
- After every merged PR: update `PROJECT_LOG.md`, post-merge cleanup, memory compression.
- The repository is **public** from creation (owner rule); never commit anything private.

## Forbidden — never without explicit owner request

- Adding a game framework or rendering library (Phaser, PixiJS…).
- Any third-party IP asset or name.
- Any backend, account system or network call in 1.0.
- Starting implementation before retro-shmup's vertical slice is done.
