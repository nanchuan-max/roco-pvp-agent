## Parent

PRD: rocom-pvp-agent (#1)

## What to build

Build the vision perception layer that reads the pre-battle team-select screen and outputs structured JSON containing the 12 pet names (my 6 + opponent's 6).

This slice is a thin vertical cut through the perception layer only — it does not need the full state machine or graph yet. The deliverable is a standalone module that:

1. Captures the game screen (using a screenshot tool).
2. Feeds the image to a vision model (e.g. `vision_analyze`) with a carefully crafted prompt that instructs the model to identify all visible pet names on the team-select screen.
3. Parses the vision output into a structured JSON list like:
   ```json
   {"my_team": ["迪莫", "喵喵", "火花", "水蓝蓝", "恶魔叮", "幽影树"],
    "opponent_team": ["火花", "水蓝蓝", "喵喵", "恶魔叮", "毛毛", "幽影树"]}
   ```
4. Includes a validation step: if the vision model returns fewer than 6 names per side, retry or flag ambiguity.

The prompt must explicitly ask the model to distinguish between the left (my) panel and the right (opponent) panel, and to list pets in the order they appear.

## Acceptance criteria

- [ ] Given a screenshot of the team-select screen, the module outputs a JSON object with exactly 6 names in `my_team` and 6 in `opponent_team`.
- [ ] The module handles cases where pet icons are partially obscured or have evolution suffixes (e.g. "迪莫·首领化" should normalize to "迪莫" for KB lookup).
- [ ] A test harness exists that can feed saved screenshots and assert the output names match expected lists.
- [ ] No state machine, no graph, no keyboard code in this slice.

## Blocked by

None — can start immediately (in parallel with slice #1).