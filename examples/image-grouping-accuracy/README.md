# Raise Grouping Accuracy Across Four Datasets

A multi-dataset objective: raise grouping accuracy on four datasets at once, with a named gate for every metric on every dataset.

- A goal like "get the accuracy higher" lets an agent stop after one dataset improves.
  - This goal names 12 gates, three metrics on each of four datasets, and is not done until all of them pass.
- The work comes from a public image-grouping challenge. The gates are the real ones from that run.
  - Every other number, path, and command is illustrative.
- Paths are written as they would appear in a target repo, under `objectives/image-grouping-accuracy/`.

## Objective Files

- `goal.md`: outcome, context refresh, strategy, success metrics, non-goals, and the 12 gates.
- `current_state.md`: the live handoff for a run in progress, with one dataset passing and a sweep running.
- `context/00_problem.md`: the grouping task, the score, and why one number is not enough.
- `context/01_constraints.md`: image-content-only inference, shard discipline, and the MLX boundary.
- `context/02_implementation_scope.md`: what the agent may edit, read, or generate, and the commands to run.
- `context/03_working_plan.md`: phase-gated research loop from baseline to promotion.
- `context/04_validation_and_handoff.md`: validation ladder, artifact contract, and state updates.
- `scripts/`: objective-local research scripts, including MLX sweeps.
- `artifacts/`: baseline, config matrices, sweep runs, visual review, and gate tables.
