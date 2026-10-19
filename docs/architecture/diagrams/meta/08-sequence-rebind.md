# Sequence diagram — meta — rebind a control (UC5)

> **Source specs**: [GDD](../../../design/game-design-document.md) §9, §12; [use-case specs](../../use-case-specifications.md) UC5
> **Related ADRs**: ADR-0006

## Context

Rebinding a keyboard or gamepad action from the options screen.

## Diagram

```mermaid
sequenceDiagram
    actor P as Player
    participant Scene as Options scene (domain)
    participant Bind as bindings (domain)
    participant In as InputPort
    participant Store as StoragePort
    P->>Scene: choose an action to rebind
    Scene->>In: captureNext()
    P->>In: press a key or button
    In-->>Scene: raw code
    Scene->>Bind: assign(action, code)
    alt code already bound to another action
        Bind-->>Scene: conflict (other action)
        Scene-->>P: ask to swap or cancel
    else free or confirmed swap
        Bind->>Store: write(bindings)
        Bind-->>Scene: updated
    end
```

## Notes

- Only keyboard and gamepad are rebindable; touch keeps its fixed pad (GDD §9).
- Reserved keys (pause, Esc) cannot be bound to a gameplay action [gen].
