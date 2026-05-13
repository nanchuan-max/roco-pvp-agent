## Parent

PRD: rocom-pvp-agent (#1)

## What to build

Build the InitGraph — a LangGraph that handles the entire pre-battle flow from team recognition to first-pet selection.

The graph must be a complete vertical slice: it takes a screenshot of the team-select screen as input, runs through perception, knowledge loading, LLM reasoning, and emits a keyboard command to select the opening pet.

Graph nodes (in order):

1. `team_perceive` — Calls the vision module from slice #2 to extract the 12 pet names from the screenshot.
2. `preload_kb` — Uses the loader from slice #1 to fetch full data for all 12 pets into memory. Builds an initial `BattleState` with:
   - `my_team`: 6 `PetState` with complete skill slots (`known=True`).
   - `opponent_team`: 6 `PetState` with empty skills and `known=False`.
3. `decide_first_pet` — LLM node with a system prompt that includes:
   - General PVP rules (MP system, energy, speed, type chart).
   - The full stats and skill pools of all 12 pets.
   - A task: "Choose the best opening pet and explain why."
   The LLM outputs a JSON with `first_pet_name` and `reasoning`.
4. `execute_select` — Calls the keyboard executor from slice #3 to press the keys that select the chosen pet as the opening fighter. Updates `BattleState.my_active_idx` and sets the chosen pet's `on_field=True`.

The graph returns the updated `BattleState` so the outer loop can hand it off to the battle phase.

## Acceptance criteria

- [ ] Running InitGraph end-to-end with a real or mocked team-select screenshot produces a `BattleState` containing 12 initialized pets.
- [ ] The LLM's chosen `first_pet_name` is one of the 6 pets in `my_team`.
- [ ] The keyboard executor (in `dry_run` mode) emits the correct key sequence for the chosen pet.
- [ ] A test exists that mocks the vision input and verifies the full graph flow without calling the real vision model.

## Blocked by

- #1 (state models + KB loader)
- #2 (vision perception for team-select)
- #3 (keyboard executor + state update)
