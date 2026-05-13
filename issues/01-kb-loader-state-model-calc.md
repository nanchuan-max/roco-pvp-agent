## Parent

PRD: rocom-pvp-agent (#1)

## What to build

Build the foundational data layer and pure-code calculators that every other slice depends on.

This slice delivers three things end-to-end:

1. **Knowledge-base loader** — Scan the local `洛克王国世界pvp知识库` directory and load the 12 pets involved in a battle into an in-memory dictionary keyed by pet name. Each entry must contain: race values (6 stats), element(s), trait name+effect, learnable skill pool, and evolution/boss-form flags.

2. **Pydantic state models** — Define `BattleState`, `PetState`, `Skill`, `TurnAction`, and `TurnLog` as Pydantic models. The design must support:
   - My team: 6 `PetState` with full known skills (`known=True`).
   - Opponent team: 6 `PetState` with empty skills and `known=False` at start.
   - Enemy belief tracking via `opponent_exposed_skills: Dict[str, List[str]]`.
   - Bench state preservation when a pet is switched out (HP and energy stay as-is).

3. **Damage & stat calculators** — Pure functions (zero LLM involvement) that implement the exact formulas from the knowledge base:
   - HP = round([1.7 × race + IV × 0.85 + 70] × (1 + nature_mod) + 100)
   - Other stats = round([1.1 × race + IV × 0.55 + 10] × (1 + nature_mod) + 50)
   - Damage = (attacker_attack × skill_power/100) × type_multiplier × defense_factor × other_modifiers
   - IVs are fixed at 60, assignable to any 3 of the 6 stats; unassigned stats treat IV as 0.

These three pieces must be wired together in a single runnable module so that a test can:
- Load a pet from the KB,
- Build a `PetState` with assigned IVs/nature,
- Compute its actual stats,
- Pick a skill,
- Compute estimated damage against another pet,
- And assert the numbers match the knowledge-base examples (e.g. 迪莫 with 固执 nature).

## Acceptance criteria

- [ ] `kb_loader` can load any pet by name from the knowledge base and return a dict with all required fields.
- [ ] Pydantic models enforce type safety and serialize/deserialize correctly.
- [ ] `calculate_actual_stats` produces the same numbers as the worked example in `战斗机制详解.md` for 迪莫 (固执, IVs on HP/物攻/速度).
- [ ] `calculate_damage` covers: physical/magic split, type chart multipliers (single and double),印记 modifiers, and连击递减.
- [ ] Unit tests exist for all three components with at least 5 distinct cases each.
- [ ] No LLM, no screenshot, no keyboard code in this slice.

## Blocked by

None — can start immediately.
