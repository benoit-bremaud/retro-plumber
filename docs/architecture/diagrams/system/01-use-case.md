# Use case diagram — system — what the player wants

> **Source specs**: [GDD](../../../design/game-design-document.md) §1, §9, §10
> **Related ADRs**: ADR-0004, ADR-0005

## Context

Who interacts with retro-plumber and to reach which goals. Grouped by domain (Play, Progress,
Settings). It does not show how the system is built (see `03-component.md`).

## Diagram

```mermaid
flowchart LR
    Player(["Player"])
    subgraph System["retro-plumber"]
        subgraph Play
            UC1(["UC1 Play a level to the flag"])
            UC2(["UC2 Pause and resume a run"])
        end
        subgraph Progress
            UC3(["UC3 Enter a name in the hall of fame"])
            UC4(["UC4 Consult the hall of fame"])
        end
        subgraph Settings
            UC5(["UC5 Rebind controls"])
            UC6(["UC6 Change language and comfort options"])
        end
    end
    Player --> UC1
    Player --> UC2
    Player --> UC3
    Player --> UC4
    Player --> UC5
    Player --> UC6
    UC2 -.->|"«extend» at playing"| UC1
    UC3 -.->|"«extend» at game-over"| UC1
    UC3 -.->|"«include»"| UC4
```

## Notes

- UC1 main success scenario: start from the title, play the level, touch the flag, see the tally
  (GDD §9). Extensions: lose a life (pit, enemy, timer) → level intro again; no life left → game
  over → UC3 if the score is in the top 10.
- UC3 «extend» UC1 at the game-over point (only when the score qualifies).
- Comfort options (UC6): reduced flash, screen shake toggle (GDD §12).
