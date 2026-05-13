## Parent

PRD: rocom-pvp-agent (#1)

## What to build

Build the outer orchestration loop that polls the game UI, detects which phase the battle is in, and dispatches to the correct graph (InitGraph or BattleGraph).

This slice is the "glue" that connects all previous slices into a continuously running agent. It is a Python `while` loop (not a LangGraph) that runs at ~500ms intervals and performs:

1. **Screenshot capture** — Grabs the current game screen.
2. **UI state detection** — Classifies the screen into one of:
   - `team_select` → trigger InitGraph (#4)
   - `first_pet_select` → wait for InitGraph to finish (or hand off if the graph handles this internally)
   - `battle_my_turn` → trigger BattleGraph (#5)
   - `battle_wait` → continue polling (opponent's turn)
   - `battle_end` → archive memory, reset state, continue polling for next match
   - `unknown` → continue polling
   
   The detector can be rule-based (look for known UI elements) or a lightweight vision prompt that classifies the screen.

3. **Graph dispatch** — When `team_select` is detected and no active battle exists, create a new `BattleState` and run InitGraph. When `battle_my_turn` is detected and a battle is active, run BattleGraph with the current state.

4. **State lifecycle** — Maintain `current_battle: Optional[BattleState]`. On `battle_end`, call `BattleMemory.archive()`, set `current_battle = None`, and log the result.

The loop must be interruptible (Ctrl+C) and must not leak resources.

## Acceptance criteria

- [ ] The loop runs continuously without crashing.
- [ ] Given a sequence of saved screenshots (team-select → battle-start → my-turn → wait → end), the detector correctly classifies each phase.
- [ ] The dispatcher invokes InitGraph exactly once per match when `team_select` is seen.
- [ ] The dispatcher invokes BattleGraph once per `battle_my_turn` event.
- [ ] On `battle_end`, the battle state is archived and the agent is ready for the next match.
- [ ] No actual keyboard events are sent during tests (all executors run in `dry_run` mode).

## Blocked by

- #4 (InitGraph must exist to be dispatched)
- #5 (BattleGraph must exist to be dispatched)
