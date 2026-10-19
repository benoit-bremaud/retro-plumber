# State machine diagram — system — game flow

> **Source specs**: [GDD](../../../design/game-design-document.md) §9
> **Related ADRs**: ADR-0005

## Context

The scenes of a session, from the title to the hall of fame. The hero's own states are in
`../gameplay/05-state-hero.md`.

## Diagram

```mermaid
stateDiagram-v2
    [*] --> Title
    Title --> LevelIntro: start
    LevelIntro --> Playing: after 120 frames
    Playing --> Paused: pause key, Esc menu, focus lost or page hidden
    Paused --> Playing: pause key, resume, tap to resume
    Playing --> Dying: pit, enemy hit when small, timer 0
    Dying --> LevelIntro: lives left
    Dying --> GameOver: no life left
    Playing --> EndSequence: flag touched
    EndSequence --> Tally: hero enters the goal house
    Tally --> LevelIntro: next level exists
    Tally --> Title: last level of the slice
    GameOver --> NameEntry: top 10 score
    GameOver --> HallOfFame: otherwise
    NameEntry --> HallOfFame
    HallOfFame --> Title
```

## Notes

- `EndSequence` runs the hero states FlagSlide then AutoWalk (see `../gameplay/05-state-hero.md`) with input locked; the timer is frozen until the tally.

- One scene active at a time; each scene is a domain object driven by `step(state, input)`
  (ADR-0005), rendered through `RenderPort`.
- The slice has one level, so `Tally` always goes back to `Title` until more levels exist.
