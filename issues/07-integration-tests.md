## Parent

PRD: rocom-pvp-agent (#1)

## What to build

Build the integration test suite that validates the core business logic across all previously built slices.

This slice is purely about testing — no new features. It must cover:

1. **Damage calculator tests** — End-to-end damage scenarios using real knowledge-base data:
   - Physical vs. magic damage split.
   - Single and double type-chart multipliers (e.g. 火 vs. 草 = 1.5×, 火 vs. 草/虫 = 2.25×).
   - Mark modifiers (攻击印记, 风起印记, 蓄势印记).
   - Defense reduction (e.g. 防御技能 70% 减伤).
   - Multi-hit skill damage decay.
   - Assertions against the worked examples in `战斗机制详解.md`.

2. **Stat calculator tests** — Verify actual stat computation matches the KB examples:
   - 迪莫 with 固执 nature, IVs on HP/物攻/速度.
   - Edge cases: zero IVs, max IVs, natures that reduce HP or speed.

3. **State merge tests** — Verify `update_state` / `merge_perception` logic:
   - Switching out a pet preserves its HP and energy.
   - Switching in a pet restores its previously saved state.
   - Opponent skill exposure appends correctly to `opponent_exposed_skills`.
   - MP deduction on faint is reflected correctly.
   - Mark layer updates (add, overwrite, stack).

4. **Belief-state tests** — Simulate a multi-turn battle and verify:
   - After the opponent uses "闪燃", the active opponent pet's `exposed_skills` contains "闪燃".
   - After a switch, the previously active opponent pet's exposed skills are retained.

All tests must use `pytest` and run in CI-friendly mode (no screenshots, no real keyboard events, no LLM calls).

## Acceptance criteria

- [ ] `pytest` runs green with 100% of the above test categories present.
- [ ] Each test category has at least 5 distinct test cases.
- [ ] Tests run in under 10 seconds total (no network, no vision, no LLM).
- [ ] A `pytest.ini` or `pyproject.toml` section configures the test runner.

## Blocked by

- #1 (damage & stat calculators must exist)
- #3 (state update logic must exist)
