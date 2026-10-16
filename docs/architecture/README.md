# Architecture — retro-plumber

UML study of brick 0 (Mermaid). The GDD is the source of truth for rules; the ADRs for
decisions; these diagrams are the visual index.

| Diagram | Question |
|---|---|
| [system/01-use-case](diagrams/system/01-use-case.md) | What does the player want to do? |
| [system/02-state-game-flow](diagrams/system/02-state-game-flow.md) | Which scenes, in which order? |
| [system/03-component](diagrams/system/03-component.md) | How do the Clean layers and ports depend on each other? |
| [gameplay/04-class-domain](diagrams/gameplay/04-class-domain.md) | Which domain types make a level run? |
| [gameplay/05-state-hero](diagrams/gameplay/05-state-hero.md) | Which states does the hero go through? |

Not drawn yet (question not open in brick 0): sequence of the fixed-step tick (copied from
retro-shmup when code starts), data flow (no personal data beyond a local name entry), render
and audio diagrams.

QA evidence (screenshots) goes under [`qa/`](qa/) once there is something to run.
