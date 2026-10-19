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
    participant In as InputPort
    participant Scene as NameEntry scene (domain)
    participant Save as save module (domain)
    participant Store as StoragePort
    FL->>Scene: step(snapshot) after GameOver
    Note over Scene: reached only when the score ranks 1-10 (GameOver scene decides)
    P->>In: letters + confirm
    FL->>In: read snapshot
    FL->>Scene: step(state, snapshot)
    Scene->>Save: insert(name, score)
    Scene-->>FL: write request (table)
    FL->>Store: write(hallOfFame)
    alt storage unavailable or full
        Store-->>FL: failure
        FL-->>Scene: kept in memory for this session
    else ok
        Store-->>FL: done
    end
    Scene-->>FL: next scene = HallOfFame
```

## Notes

- Name length and charset are not yet specified in the GDD (open point, own PR) [gen]; a name is not translated.
- The domain never calls the port: the app performs the write (ADR-0006).
- Storage failure never blocks the flow: the table lives in memory until the page closes.
