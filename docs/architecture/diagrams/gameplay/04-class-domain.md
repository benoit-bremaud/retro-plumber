# Class diagram — gameplay — domain model

> **Source specs**: [GDD](../../../design/game-design-document.md) §4–§8
> **Related ADRs**: ADR-0002, ADR-0006

## Context

The main domain types of the vertical slice and their relations. Rendering, audio and storage
are not shown (ports, see `../system/03-component.md`).

## Diagram

```mermaid
classDiagram
    class LevelRun {
        +step(input) void
        +score int
        +coins int
        +timer int
    }
    class Level {
        +width int
        +tileAt(col, row) TileKind
    }
    class TileKind {
        <<union>>
        Empty
        Ground
        HardBlock
        Brick
        ItemBlock
        HiddenBlock
        Pipe
        FlagPole
    }
    class Camera {
        +x int
        +follow(hero) void
    }
    class Hero {
        +x int
        +y int
        +vx int
        +vy int
        +size HeroSize
        +state HeroState
    }
    class MovementTables {
        <<constants>>
        caps
        accelerations
        jumpSpeeds
        gravities
    }
    class HeroPhysics {
        +stepHorizontal(hero, input) void
        +stepVertical(hero, input) void
    }
    class TileCollision {
        +resolveX(body, level) void
        +resolveY(body, level) void
    }
    class EnemyPool {
        +activate(spawn) void
        +stepAll(level) void
    }
    class EnemyKind {
        <<union>>
        Walker
        ShelledWalker
        Shell
    }
    class ScoreChain {
        +onStomp() int
        +reset() void
    }
    LevelRun *-- Level
    LevelRun *-- Hero
    LevelRun *-- Camera
    LevelRun *-- EnemyPool
    LevelRun *-- ScoreChain
    Level --> TileKind
    HeroPhysics ..> MovementTables
    HeroPhysics ..> Hero
    TileCollision ..> Level
    EnemyPool --> EnemyKind
```

## Notes

- All positions and speeds are fixed-point integers (GDD §4); no float in the domain.
- `TileKind` and `EnemyKind` are discriminated unions with behaviour tables: a new enemy or tile
  is a new entry, not a new branch in the loop (open/closed).
- `EnemyPool` is preallocated (no allocation in the loop).
- `RandomSource` (seeded) is omitted: the slice has no random rule yet (YAGNI).
