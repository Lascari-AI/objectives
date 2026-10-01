setup-objective is an authoring and maintenance skill for the agent already working in the repository. It gives that agent a file convention, writing guidance, and two helper scripts. It does not launch a separate worker.

## Inputs and Authority

The agent starts from what already exists in the repository.

- It reads the user's outcome, existing objectives, repository instructions, and any active state file.

- It preserves established objective names and unrelated work.

- It treats a top-level CURRENT_STATE.md as legacy index material.

  - The agent moves only the relevant state into the objective's own current_state.md.

  - It leaves unrelated content alone unless the user asks for a rewrite.

- The skill is defined in .codex/skills/setup-objective/SKILL.md.

  - Detailed authoring rules live in .codex/skills/setup-objective/references/objective_authoring.md.

## Authoring Workflow

The agent follows the same steps for every objective.

1. Inspect existing objectives and decide whether the request is new work or a continuation.

2. Scaffold a new bundle with a stable lowercase hyphenated slug. Then replace every template bullet with project-specific inputs, commands, artifacts, and gates.

3. Keep goal.md under 4,000 characters and move background, research notes, and phase plans into context/.

4. For runs longer than a few minutes, define logs, status, checkpoints, resume, and active-run handoff before the run starts.

5. Update current_state.md at the start, after milestones, before handoff or compaction, and before the final response.

## Research and Optimization

For optimization work, the skill treats experiment layout as part of the objective design.

- The working plan reproduces the baseline before it changes behavior.

- Candidate variables go in a `config_matrix.csv` or an equivalent manifest.

- Cheap proxy sweeps only screen candidates. Selected anchors, Pareto candidates, and near-misses get full validation.

  - Anchors and near-misses keep the search from picking only the top proxy score.

- Visual findings become deployable feature hypotheses.

  - Truth labels, filenames, shard identity, and hand-reviewed labels support measurement only. They never become production rules.

- Strategy guidance lives in .codex/skills/setup-objective/references/known_optimization_strategies/README.md.

  - Long-run requirements live in .codex/skills/setup-objective/references/long_running_handoff_and_scripts.md.

## Outputs

The result is an objective bundle that a fresh agent can execute.

- The bundle has a bounded outcome, explicit stopping criteria, executable phases, and an honest current state.

- If setup leaves open questions that affect validity, the agent raises them before execution instead of filling them with assumptions.

## Host Integration

The skill is plain files, so any agent that reads the repository can use it.

- A host that discovers SKILL.md packages exposes setup-objective through its normal skill mechanism.

- You can also give the agent the path to SKILL.md directly.

- The host owns discovery, permissions, continuation, and budgets.

- Codex `/goal` is one consumer of goal.md.

  - The validator enforces the 4,000-character limit.

  - Neither helper creates a host goal or changes its status.
