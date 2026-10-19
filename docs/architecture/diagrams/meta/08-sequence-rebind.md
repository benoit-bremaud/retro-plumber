# Sequence diagram — meta — rebind a control (UC5)

> **Source specs**: [GDD](../../../design/game-design-document.md) §9, §12; [use-case specs](../../use-case-specifications.md) UC5
> **Related ADRs**: ADR-0006

## Context

Rebinding a keyboard or gamepad action from the options screen.

## Diagram

```mermaid
sequenceDiagram
    actor P as Player
    participant In as InputPort
    participant FL as FrameLoop (app)
    participant Scene as Options scene (domain)
    participant Bind as bindings (domain)
    participant Store as StoragePort
    P->>Scene: choose an action to rebind (menu step)
    Scene-->>FL: state = waiting for capture
    P->>In: press a key or button
    FL->>In: read snapshot
    In-->>FL: InputSnapshot (raw code)
    FL->>Scene: step(state, snapshot)
    alt Esc or cancel
        Scene-->>FL: nothing changes
    else reserved key
        Scene-->>FL: rejected, message
    else code bound to another action
        Scene-->>FL: ask to swap or cancel
    else free or confirmed swap
        Scene->>Bind: assign(action, code)
        Bind-->>Scene: updated bindings
        Scene-->>FL: updated bindings + write request
        FL->>Store: write(bindings)
        alt storage unavailable
            Store-->>FL: failure, applies for this session only
        end
    end
```

## Notes

- Only keyboard and gamepad are rebindable; touch keeps its fixed pad (GDD §9).
- The domain never reads the port: the app passes the raw code inside the snapshot and performs
  the write (ADR-0006). Menu navigation is omitted for brevity.
- Reserved keys (pause, Esc) cannot be bound to a gameplay action [gen].
