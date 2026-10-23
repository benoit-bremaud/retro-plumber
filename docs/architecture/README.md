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
| [gameplay/06-sequence-tick](diagrams/gameplay/06-sequence-tick.md) | How does one fixed step run (UC1, UC2)? |
| [gameplay/10-sequence-collisions](diagrams/gameplay/10-sequence-collisions.md) | How are a stomp and a block hit resolved (UC1)? |
| [gameplay/11-sequence-item-pickup](diagrams/gameplay/11-sequence-item-pickup.md) | How does an item emerge and get collected (UC1)? |
| [gameplay/12-sequence-flag-tally](diagrams/gameplay/12-sequence-flag-tally.md) | How does the level end (UC1)? |
| [gameplay/13-sequence-death-retry](diagrams/gameplay/13-sequence-death-retry.md) | What follows a death (UC1)? |
| [gameplay/14-sequence-pause-resume](diagrams/gameplay/14-sequence-pause-resume.md) | How do pause and auto-pause work (UC2)? |
| [meta/07-sequence-name-entry](diagrams/meta/07-sequence-name-entry.md) | How is a name saved (UC3)? |
| [meta/08-sequence-rebind](diagrams/meta/08-sequence-rebind.md) | How is a control rebound (UC5)? |
| [meta/09-data-flow-persistence](diagrams/meta/09-data-flow-persistence.md) | What is stored, where? |
| [meta/15-class-meta](diagrams/meta/15-class-meta.md) | Which types hold scenes, save, bindings, texts? |
| [meta/16-data-flow-tick](diagrams/meta/16-data-flow-tick.md) | What crosses the ports in a frame? |

Use-case specifications: [use-case-specifications.md](use-case-specifications.md). Traceability:
[traceability-matrix.md](traceability-matrix.md).

Not drawn yet: render and audio diagrams; options-screen state (gap G3 of the traceability matrix).

QA evidence (screenshots) goes under [`qa/`](qa/) once there is something to run.
