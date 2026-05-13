## Parent

PRD: rocom-pvp-agent (#1)

## What to build

Build the action execution layer, the persistent memory layer, and the state-merge logic that bridges perception and decision.

This slice delivers three end-to-end capabilities:

1. **Keyboard executor** — A thin wrapper around `pynput` or `pydirectinput` that accepts a structured decision JSON and sends the correct key sequence:
   - `"action_type": "skill", "slot": 1` → press `1`
   - `"action_type": "switch"` → press `E`, then navigate to target pet via arrow keys / number keys, then confirm
   - `"action_type": "gather"` → press `R`
   - `"action_type": "flee"` → press `Esc`
   The executor must include a `dry_run` mode for testing that prints keys instead of sending them.

2. **Memory persistence** — A `BattleMemory` class that appends each turn's complete `BattleState` to a JSONL file (`battle_logs/{battle_id}.jsonl`). It also supports loading the most recent N turns for LLM context and archiving completed battles.

3. **State update logic** — A `merge_perception(battle_state: BattleState, perceived_json: dict)` function that:
   - Updates HP and energy from the vision-derived JSON.
   - Preserves bench-pet state when switching (HP and energy remain unchanged for the pet that left the field).
   - Appends a new `TurnLog` with the actions from the previous turn.
   - Updates `opponent_exposed_skills` when the opponent uses a newly seen skill.

The three pieces must be wired together in a mini-demo script that:
- Creates a `BattleState`,
- Simulates a "perceived JSON" showing HP/energy changes and an opponent skill usage,
- Calls `merge_perception`,
- Saves the updated state to JSONL,
- And prints the keyboard keys that would have been pressed for a mock decision.

## Acceptance criteria

- [ ] Keyboard executor can be tested in `dry_run` mode with no actual key events sent.
- [ ] `BattleMemory` writes valid JSONL that can be read back and round-trips through `BattleState.model_dump()`.
- [ ] `merge_perception` correctly preserves bench pet HP/energy when a switch occurs.
- [ ] `merge_perception` correctly adds a newly seen opponent skill to `opponent_exposed_skills`.
- [ ] Unit tests cover: dry-run keyboard sequences, JSONL round-trip, state merge with/without switches.

## Blocked by

- #1 (needs `BattleState`, `PetState`, `TurnLog` models and `Skill` structure)
