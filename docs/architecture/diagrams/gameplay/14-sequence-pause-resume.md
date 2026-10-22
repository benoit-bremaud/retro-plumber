# Sequence diagram — gameplay — pause, auto-pause and resume (UC2)

> **Source specs**: [GDD](../../../design/game-design-document.md) §8, §9; [use-case specs](../../use-case-specifications.md) UC2
> **Related ADRs**: ADR-0005, ADR-0006

## Context

Detail of the pause branch drawn in short in [06](06-sequence-tick.md): who detects the trigger,
who decides, and how the 43-frame lock behaves. Pause is an input event, so a replay script
contains it.

## Diagram

```mermaid
sequenceDiagram
    actor P as Player
    participant Br as Browser events
    participant In as InputPort adapter
    participant FL as FrameLoop (app)
    participant LR as Game / LevelRun (domain)
    participant Out as RenderPort
    alt key, Start or touch button
        P->>In: Enter, P, Start or pause button
        In->>In: pause bit set in the next snapshot
    else focus lost or page hidden
        Br->>In: blur or visibilitychange
        In->>In: auto-pause bit set in the next snapshot
    end
    FL->>In: read snapshot
    In-->>FL: InputSnapshot (pause toggle bit, auto-pause bit)
    FL->>LR: step(state, snapshot)
    alt auto-pause bit and playing
        LR->>LR: paused = true, pauseLockFrames = 43 (lock bypassed)
    else toggle bit and pauseLockFrames > 0
        LR->>LR: toggle ignored
    else toggle bit, lock is zero and playing
        LR->>LR: paused = true, pauseLockFrames = 43
    end
    LR-->>FL: new state
    loop each browser frame while paused
        loop each accumulated fixed step
            FL->>In: read snapshot
            In-->>FL: InputSnapshot
            FL->>LR: step(state, snapshot)
            Note over LR: all timers frozen except pauseLockFrames, which counts down
            opt toggle pressed or tapped to resume, lock is zero and no auto-pause bit
                LR->>LR: toggle accepted, paused = false, pauseLockFrames = 43
                Note over LR: remaining accumulated steps of this frame run as playing (06)
            end
            LR-->>FL: new state
        end
        FL->>Out: draw(state), once per frame after the step loop (06), pause view while paused
    end
    Note over FL,LR: the outer loop ends on the frame whose step accepted the toggle, the run continues
```

## Notes

- `Game / LevelRun` is the same merged participant as in [06](06-sequence-tick.md): `Game` dispatches `step` to the active scene, which delegates to `LevelRun` (12 and 13 draw the two apart).
- Tags. The 43-frame lock: **[src]**. Pause keys, auto-pause on blur or hidden page, "tap to
  resume" overlay, manual resume only: **[gen]** (GDD §9, Q7).
- The adapter only turns a browser event into a bit of the snapshot; deciding what the bit means
  (toggle, ignore) is a domain rule. This keeps replays exact (ADR-0005).
- The snapshot carries two bits: the toggle (keys, Start, touch button) and an auto-pause bit that can only pause. So a blur while paused never resumes, and a blur inside the lock still pauses (the lock guards manual toggles only) **[gen]**. Re-arming the lock on resume mirrors the pause **[gen]**.
- An auto-pause while already paused changes nothing; resuming is never automatic (UC2 1a).
- A manual toggle inside the lock is ignored (UC2 2a); because the lock keeps counting during pause,
  the first resume press after a pause is accepted once it reaches zero (a press in the first 43 frames is ignored).
- Esc opens the menu rather than pausing; where options are reached is OQ2 and is not drawn.
