# Use-case specifications — retro-plumber

**Version:** 0.1 — 2026-10-19 (brick 4, issue #6). **Status:** Accepted (owner, 2026-10-19).

Cockburn-style (*Writing Effective Use Cases*, fully dressed) specifications of the six use cases
of [`diagrams/system/01-use-case.md`](diagrams/system/01-use-case.md). Every use case is an
**actor-initiated goal**. Rules and numbers come from the [GDD](../design/game-design-document.md)
v1.0 and keep its tags: **[src]** public source, **[int]** our derivation, **[gen]** genre
knowledge or our own choice, not checked against a source. Coin, brick, growth-item and time-bonus values stay **[gen]**
(GDD §8, unverified); the stomp chain and flag bands are [src]. Traceability to the other diagrams: [`traceability-matrix.md`](traceability-matrix.md).

Common to all use cases: primary actor **Player**; scope **retro-plumber (browser game)**; no
network, no account (GDD §12); every player-facing string comes from the EN/FR catalogue.

## Relations

| Relation | Meaning |
|---|---|
| UC2 «extend» UC1 | Pause may interrupt UC1 at any point of the *Playing* scene (extension point `playing`). |
| UC3 «extend» UC1 | Name entry is added at the game-over point when the score is in the top 10 (extension point `game-over`). |
| UC3 «include» UC4 | After the name is saved, the hall of fame is shown (the whole of UC4, from step 1). |
| UC1 → UC4 | When the game-over score does not qualify, UC1 ends directly in UC4 (GDD §9). |
| UC5, UC6 | Independent goals, reached from the options screen. |

## UC1 — Play a level to the flag

- **Level:** user goal. **Trigger:** the player starts a game from the title.
- **Preconditions:** title scene shown; assets and catalogue loaded.
- **Minimal guarantee:** the session state (score, coins, lives) is never corrupted; closing the page loses only the current run.
- **Success guarantee:** the flag is reached, the tally is added to the score, the game returns to the title (slice: one level, GDD §10).
- **Main scenario:**
  1. Player starts the game; the system shows the level intro (name, lives) for 120 frames [gen].
  2. The system starts the run: timer at 400 units [gen] (chosen, GDD Q5/Q11), 1 unit = 24 frames [src].
  3. Player moves, runs and jumps (momentum, variable jump height [src]); the system applies the movement tables (GDD §4) every 1/60 s.
  4. Player stomps enemies, breaks bricks, bumps item blocks, collects coins and the growth item; the system scores each event (GDD §8) and grows the hero.
  5. Player touches the flag pole; the system locks input, scores the grab band [src/int], slides the hero down, walks him into the goal house.
  6. The system adds 50 points per remaining timer unit [gen] and shows the tally.
  7. The system returns to the title.
- **Extensions:**
  - 3a. Player pauses → UC2.
  - 3b. Timer reaches 100 units → warning jingle, faster music [src].
  - 4a. Hero hit while big → small, 158 frames of invulnerability [int]; the run continues.
  - 4b. Hero hit while small, falls into a pit, or timer reaches 0 → death hop (−4 px/frame [src]); lives −1.
    - 4b1. Lives left → back to step 1 with a fresh run, score/coins/lives kept (class diagram 04) [gen].
    - 4b2. No life left → game over; if the score is in the top 10 → UC3, otherwise → UC4.
  - 4c. Growth item collected during invulnerability → hero becomes big, invulnerability ends [gen].
  - 4d. 100 coins collected → +1 life, counter reset [src]; stomp chain reaching its end → +1 life [src].
- **Special requirements:** deterministic replay of the input script (ADR-0005); no allocation in the loop; game-feel effects (blink, shake) toggleable (GDD §12).
- **Open point:** a golden-trajectory test measures the real margin of pit 3 (GDD §10).

## UC2 — Pause and resume a run

- **Level:** user goal (optional behaviour inserted into UC1). **Trigger:** the player presses Enter, P or Start; or the page loses focus / becomes hidden (auto-pause).
- **Preconditions:** a level is in the *Playing* scene.
- **Minimal guarantee:** the simulation state is unchanged by the pause.
- **Success guarantee:** the run continues exactly where it stopped.
- **Main scenario:**
  1. Player triggers pause; the system freezes the simulation and shows the pause view.
  2. Player triggers pause again (or taps to resume on touch); the system resumes the simulation.
- **Extensions:**
  - 1a. Auto-pause (focus lost, page hidden) → same state; resume is manual, never automatic [gen].
  - 1b. Esc → the pause view also offers the menu (UC5, UC6 reachable from it, see OQ2).
  - 2a. Re-trigger inside the 43-frame lock [src] → ignored; the lock keeps counting during pause.
- **Special requirements:** pause is an input event recorded in the replay; all timers stop except the pause lock (GDD §8, §9).

## UC3 — Enter a name in the hall of fame

- **Level:** user goal. **Trigger:** game over with a score in the top 10 (extends UC1).
- **Preconditions:** game-over scene shown; score ranks 1–10.
- **Minimal guarantee:** the table stays valid (≤ 10 entries, sorted); a refused or failed save never blocks the flow.
- **Success guarantee:** the name and score are in the table and persisted locally.
- **Main scenario:**
  1. The system asks for a name and shows the rank.
  2. Player enters letters and confirms.
  3. The system inserts the entry, writes the table through the storage port, and shows the hall of fame (UC4).
- **Extensions:**
  - 2a. Player confirms an empty name → default name [gen].
  - 3a. Storage unavailable or full → the table is kept in memory for this session; the flow continues; a notice may be shown [gen].
- **Special requirements:** the name is the only personal datum, kept in the player's browser only (data-flow `09`).

## UC4 — Consult the hall of fame

- **Level:** user goal. **Trigger:** end of UC3, or game over with a non-qualifying score (GDD §9).
- **Preconditions:** none beyond a loaded game.
- **Minimal guarantee:** an empty or unreadable table shows defaults, never an error screen.
- **Success guarantee:** the top 10 is shown, the game returns to the title.
- **Main scenario:**
  1. The system loads the top 10 and displays rank, name, score.
  2. Player dismisses the screen; the system returns to the title.
- **Extensions:**
  - 1a. No saved table → default entries [gen].
  - 1b. Corrupt record → replaced by defaults (data-flow `09`).
- **Open point (OQ1):** GDD §9 reaches the hall of fame only through game over; direct access from the title is not specified (needs a GDD revision, own PR).

## UC5 — Rebind controls

- **Level:** user goal. **Trigger:** the player opens the options screen and picks "controls".
- **Preconditions:** options screen reachable (OQ2).
- **Minimal guarantee:** every gameplay action keeps at least one binding; invalid captures are rejected.
- **Success guarantee:** the new binding is active at once and persisted.
- **Main scenario:**
  1. Player selects an action to rebind; the system waits for the next key or button.
  2. Player presses a key (keyboard) or button (gamepad).
  3. The system assigns it, saves the bindings, and shows the updated list.
- **Extensions:**
  - 2a. Code already bound to another action → the system asks to swap or cancel.
  - 2b. Reserved key (pause, Esc) [gen] → rejected with a message.
  - 2c. Player cancels → nothing changes.
  - 3a. Storage unavailable → the binding applies for this session only.
- **Special requirements:** touch pad is not rebindable (GDD §9); left + right together are ignored (GDD §9).

## UC6 — Change language and comfort options

- **Level:** user goal. **Trigger:** the player opens the options screen and picks language or comfort.
- **Preconditions:** options screen reachable (OQ2).
- **Minimal guarantee:** options always hold a valid value (defaults otherwise).
- **Success guarantee:** the choice applies immediately and is persisted.
- **Main scenario:**
  1. Player selects a language (EN, FR) or toggles a comfort option (reduced flash, screen shake, blink).
  2. The system applies it at once (catalogue switch, effect flags) and saves the options.
- **Extensions:**
  - 2a. Storage unavailable → applies for this session only.
- **Special requirements:** every game-feel effect is toggleable and respects photosensitivity options (GDD §12).

## Open questions (not decided here; GDD revision in its own PR)

| # | Question | Impact |
|---|---|---|
| OQ1 | Direct access to the hall of fame from the title? | UC4 trigger |
| OQ2 | Where is the options screen reached (title menu, Esc menu, both)? | UC2, UC5, UC6 triggers; game-flow diagram has no options scene |
| OQ3 | Coin, brick, growth-item and time-bonus values of GDD §8 are [gen], unverified | UC1 step 6 numbers |
