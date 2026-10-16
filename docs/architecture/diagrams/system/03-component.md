# Component diagram — system — Clean layers and ports

> **Source specs**: [CLAUDE.md](../../../../CLAUDE.md) § Architecture
> **Related ADRs**: ADR-0001, ADR-0006

## Context

The structural boundaries: the pure domain, the ports it declares, the browser adapters and the
composition root. Dependencies point inward only.

## Diagram

```mermaid
flowchart TB
    subgraph App["src/app — composition root"]
        Main["main: builds adapters, injects ports"]
        Loop["FrameLoop: fixed 60 Hz accumulator"]
    end
    subgraph Adapters["src/adapters — browser"]
        Canvas["CanvasRenderer"]
        Keys["KeyboardGamepadTouchInput"]
        Audio["WebAudioSynth"]
        Store["LocalStorageStore"]
        Clock["PerformanceClock"]
    end
    subgraph Domain["src/domain — pure TypeScript"]
        Ports["ports: RenderPort, InputPort, AudioPort, StoragePort, ClockPort"]
        Game["game, scenes"]
        Physics["physics"]
        Level["level"]
        Entities["entities"]
    end
    Main --> Loop
    Main --> Canvas
    Main --> Keys
    Main --> Audio
    Main --> Store
    Main --> Clock
    Loop --> Game
    Canvas -->|"implements"| Ports
    Keys -->|"implements"| Ports
    Audio -->|"implements"| Ports
    Store -->|"implements"| Ports
    Clock -->|"implements"| Ports
    Game --> Ports
    Game --> Physics
    Game --> Level
    Game --> Entities
    Entities --> Physics
```

## Notes

- No arrow from `Domain` to `Adapters` or `App`: enforced by `tsconfig.domain.json` (no DOM lib)
  and an ESLint import boundary (ADR-0006).
- Each port is small and client-specific (ISP); tests replace them with fakes (ADR-0005).
- The game reads input as a snapshot per step; it never calls the clock directly.
