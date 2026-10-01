# Reduce Parser Runtime

A simple single-metric objective: make the parser faster on one fixed fixture without changing its output or its memory use.

- The project, fixture, commands, and numbers are illustrative.
  - Replace them with verified values from your own project before running.
- Keep durable handoff notes in `current_state.md`, not in a top-level `CURRENT_STATE.md`.
- Paths are written as they would appear in a target repo, under `objectives/parser-benchmark/`.

## Objective Files

- `goal.md`: outcome, context refresh, strategy, success metrics, non-goals, and completion criteria.
- `current_state.md`: the live handoff for a run in progress.
- `context/00_problem.md`: why the parser is slow and what a win is worth.
- `context/01_constraints.md`: hard rules, forbidden shortcuts, and gates.
- `context/02_implementation_scope.md`: files the agent may edit, read, or generate, and the commands to run.
- `context/03_working_plan.md`: phase-gated plan from baseline to final decision.
- `context/04_validation_and_handoff.md`: validation ladder, artifact contract, and state updates.
- `scripts/`: objective-local benchmark and comparison scripts.
- `artifacts/`: baseline, profiles, and one folder per experiment run.
