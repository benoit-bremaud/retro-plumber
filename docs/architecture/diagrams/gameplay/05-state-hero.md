# State machine diagram — gameplay — hero

> **Source specs**: [GDD](../../../design/game-design-document.md) §4.2, §4.3
> **Related ADRs**: ADR-0002

## Context

Two orthogonal regions: the hero's size (power-up state) and its motion state. The motion
values of each state are the GDD §4 tables.

## Diagram

```mermaid
stateDiagram-v2
    state Hero {
        state Size {
            [*] --> Small
            Small --> Big: growth item
            Big --> SmallInvulnerable: hit
            SmallInvulnerable --> Small: after 2 s
            Small --> Dead: hit
        }
        --
        state Motion {
            [*] --> Grounded
            Grounded --> Rising: new jump press
            Rising --> Falling: jump released after 1 px, or vy at or above 0
            Grounded --> Falling: walks off a ledge
            Falling --> Grounded: lands
            Rising --> Falling: head bump
            Falling --> Rising: stomp bounce
            Falling --> [*]: falls into a pit, life lost
        }
    }
```

## Notes

- `Rising` uses the "held" gravity of the take-off row, `Falling` the "released" one (GDD §4.2);
  the pair chosen at take-off is kept until the next jump, even across a stomp bounce.
- `Grounded` covers walk, run and skid; they are speed ranges, not separate states (GDD §4.1).
- Shooter size is out of the slice (GDD §4.3).
