## Parent

PRD: rocom-pvp-agent (#1)

## What to build

Build the BattleGraph — a LangGraph that handles one complete turn of the PVP battle: perceive the screen, update state, reason with the LLM, and execute the chosen action.

This is the core decision loop of the agent. The graph receives:
- A screenshot of the current battle UI (when it's my turn)
- The current `BattleState` (carried forward from the previous turn or from InitGraph)
- The pre-loaded knowledge base

Graph nodes (in order):

1. `perceive` — Calls vision analysis to parse the battle screenshot into structured JSON (my active pet HP/energy/marks, opponent active pet HP/energy/marks, bench alive counts, any visible status effects).
2. `update_state` — Merges the perceived JSON into the existing `BattleState` using the merge logic from slice #3. Updates HP/energy, handles switches, appends the previous turn's actions to history, and updates opponent belief state if new skills were used.
3. `decide` — LLM node with:
   - System prompt containing general PVP rules (MP system, energy, speed, marks, type chart).
   - Current battle state as text + the last 5 turns of history.
   - Available Tools:
     - `query_kb` (look up skill/pet/rule details on demand)
     - `calculate_damage` (estimate damage for a given skill against the opponent)
     - `calculate_actual_stats` (compute real stats if needed)
   - Output format: JSON with `action_type`, `detail`, `slot` (for skills), `reasoning`, `expected_outcome`.
4. `execute` — Sends the keyboard command via the executor from slice #3.

After execution, the outer loop will wait for the opponent's turn and then invoke BattleGraph again with the updated state.

## Acceptance criteria

- [ ] BattleGraph can be invoked end-to-end with a mocked battle screenshot and mocked vision output.
- [ ] The `decide` node's LLM receives the correct context: current state + last 5 turns + available tools.
- [ ] Tool calls from the LLM are correctly routed: `calculate_damage` returns numeric estimates, `query_kb` returns knowledge strings.
- [ ] The `execute` node emits the correct key sequence in `dry_run` mode for at least these action types: skill (1-4), switch, gather.
- [ ] A test exists that runs the full graph with mocked inputs and asserts the final decision JSON schema is valid.

## Blocked by

- #1 (state models + calculators)
- #3 (keyboard executor + memory + state update)
