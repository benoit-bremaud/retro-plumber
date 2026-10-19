# Sequence diagram — meta — name entry in the hall of fame (UC3)

> **Source specs**: [GDD](../../../design/game-design-document.md) §9; [use-case specs](../../use-case-specifications.md) UC3, UC4
> **Related ADRs**: ADR-0006

## Context

From the game-over scene to the saved top 10, through the storage port.

## Diagram

```mermaid
sequenceDiagram
    actor P as Player
    participant FL as FrameLoop (app)
    participant Scene as NameEntry scene (domain)
    participant Save as save module (domain)
    participant Store as StoragePort
    FL->>Scene: step(snapshot) after GameOver
    Scene->>Save: qualifies(score, table)
    Save-->>Scene: yes (rank n)
    P->>Scene: letters + confirm (via InputPort)
    Scene->>Save: insert(name, score)
    Save->>Store: write(hallOfFame)
    alt storage unavailable or full
        Store-->>Save: failure
        Save-->>Scene: kept in memory for this session
    else ok
        Store-->>Save: done
    end
    Scene-->>FL: next scene = HallOfFame
```

## Notes

- Name length and allowed characters are a catalogue/save rule (a name is not translated).
- Storage failure never blocks the flow: the table lives in memory until the page closes.
