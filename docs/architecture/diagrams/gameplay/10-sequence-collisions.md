# Sequence diagram — gameplay — stomp and block hit (UC1)

> **Source specs**: [GDD](../../../design/game-design-document.md) §4.2, §5, §6, §8; [use-case specs](../../use-case-specifications.md) UC1 steps 3–4, 4d
> **Related ADRs**: ADR-0002, ADR-0005, ADR-0006

## Context

What happens inside one `step` when the hero meets an enemy or a block. It zooms into the
"physics, collision, entities, contacts" box of [06](06-sequence-tick.md). Everything shown runs
in the domain; the only outgoing data is the list of events the app forwards to `AudioPort`.

## Diagram

```mermaid
sequenceDiagram
    participant FL as FrameLoop (app)
    participant LR as LevelRun (domain)
    participant HP as HeroPhysics
    participant TC as TileCollision
    participant BB as BlockBump
    participant EP as EnemyPool
    participant Ct as Contacts
    participant H as Hero
    participant Sess as Session
    participant Snd as AudioPort
    FL->>LR: step(state, snapshot)
    LR->>HP: stepHorizontal(hero, input)
    LR->>TC: resolveX(hero, grid)
    LR->>HP: stepVertical(hero, input)
    LR->>TC: resolveY(hero, grid)
    opt head bump (block above the hero's centre)
        TC-->>LR: block hit (col, row), vy set to +1 px/frame
        alt brick and hero is Big
            LR->>LR: tile becomes Empty, event brick-broken
            LR->>Sess: add 50 points
        else brick and hero is Small
            LR->>BB: start(col, row), offset 18 px
            Note over LR: the brick stays a Brick after the bump
            BB->>EP: bump check on enemies standing on the block
            EP-->>LR: those enemies flipped and killed
        else unused item block or hidden coin block
            LR->>BB: start(col, row), offset 18 px (Small) or 4 px (Big)
            LR->>LR: block becomes UsedBlock, growth item: ItemPool.activate, coin (item or hidden block): credited at the hit (see 11)
            BB->>EP: bump check on enemies standing on the block
            EP-->>LR: those enemies flipped and killed
        else hard block, pipe or already UsedBlock
            Note over LR: no reaction, tile unchanged
        end
    end
    LR->>EP: stepAll(grid)
    LR->>Ct: heroVsEnemies(hero, enemies), index order
    alt no enemy overlaps the hero
        Note over Ct: nothing happens
    else hero overlaps an enemy, falling and feet above its centre (stomp)
        Ct->>EP: Grubbit flattened (30 frames) or Dentshell becomes a still shell
        alt stompChain is within the ten-step table
            Ct->>Sess: add chainPoints(stompChain)
            Ct->>H: stompChain + 1
        else the table is exhausted
            Ct->>Sess: lives + 1 (no points)
        end
        Note over Ct: hero vy set to -4 px/frame, gravity pair unchanged
    else hero overlaps an enemy from the side or below
        alt hero is Big or Small (not invulnerable)
            Ct-->>LR: hero hit (see diagrams 13 and 05)
        else hero is SmallInvulnerable
            Note over Ct: contact ignored, hero unchanged (05)
        end
    end
    LR-->>FL: new state + events (stomp, brick-broken, bump...)
    FL->>Snd: play(events)
```

## Notes

- Tags. Head bump speed +1 px/frame, bump offsets 4 / 18 px, stomp bounce −4 px/frame, chain
  points and the +1 life at the chain's end: **[src]** (GDD §5, §4.2, §8). Stomp definition,
  contact test every frame, index-order resolution, 30-frame flattening, 50 points per brick:
  **[gen]** (GDD §5, §6, §8); brick points are unverified (OQ3).
- Hidden coin block (GDD §7, §10, [gen]): hit from below it behaves like a coin item block and becomes `UsedBlock`.
- Axis-separated movement (X then resolve, Y then resolve) is why `TileCollision` is called
  twice (GDD §5); the 4 px corner tolerance [gen] lives inside `resolveY`.
- Stomping a kicked shell stops it (GDD §6); not redrawn. A kicked shell reuses the same chain table (GDD §8, [gen]); its contacts are not redrawn.
- `stompChain` lives on the hero and resets on landing (04). Pools are preallocated: `activate`
  and the flattening reuse slots, nothing is allocated in the step.
- The domain never calls `AudioPort`: events are returned and forwarded by the app (ADR-0006).
