Create a small objective before you use the format for an expensive run. This guide covers setup, a worked example, and resuming a run. File ownership and completion rules live in Objective Bundle and Lifecycle.

The example is illustrative. Replace its benchmark, files, and thresholds with verified values from your own project.

## Make the Skill Available

Clone the objectives repository into a tools directory.

- You need Git and Python 3 for the two helpers.

- Keep the skill's references and scripts together.

```bash
git clone https://github.com/Lascari-AI/objectives.git
cd objectives
python3 .codex/skills/setup-objective/scripts/create_objective.py --help
```

- For any host, give your agent the absolute path to this checkout's `.codex/skills/setup-objective/SKILL.md` and ask it to use the skill in your target repository.

  - This start does not depend on automatic skill discovery.

- For repository-local Codex discovery, copy the whole setup-objective folder into your project's `.codex/skills/` directory.

  - Follow your client's skill reload behavior.

  - Do not copy this repository's `.codex/config.toml`.

## Define the Stopping Criteria

Write the stopping criteria before you scaffold.

- Suppose a parser has a repeatable benchmark and an output-equivalence test.

- Name every gate the result must clear.

  - Median runtime drops by at least 15 percent on a fixed fixture.

  - Parsed output stays identical.

  - Peak memory stays within the measured baseline.

- A single target such as "make it faster" lets the agent stop at the first improvement.

- Name the benchmark command, fixture fingerprint, baseline measurements, and the files the agent may edit.

  - If any are unknown, make finding them the first phase instead of inventing values.

## Create the Bundle

Run the scaffolder from your target project's root. Replace the absolute path with your checkout path.

```bash
python3 /absolute/path/to/objectives/.codex/skills/setup-objective/scripts/create_objective.py \
  parser-benchmark --title "Reduce Parser Runtime"
```

- The script prints `Created objective bundle: objectives/parser-benchmark`.

  - It creates eight Markdown files plus `examples/.gitkeep`.

- Review the folder, then have the agent replace every template bullet with the agreed problem, scope, evidence, and gates.

## Write the Goal

The goal holds the stopping criteria and points to the files that hold everything else.

```xml
<goal>
- Reduce parser benchmark median runtime by at least 15 percent without changing parsed output or increasing peak memory.
</goal>
<context_refresh>
- Reread objectives/parser-benchmark/goal.md.
- Reread objectives/parser-benchmark/current_state.md.
- Reread the relevant objectives/parser-benchmark/context/*.md files.
- Reread these files at the start of work and after every compaction or resume.
</context_refresh>
<working_strategy>
- Reproduce the fixed baseline, profile the bottleneck, change one mechanism, and compare repeated runs.
</working_strategy>
<success_metrics>
- Lower median runtime with output equivalence and peak memory within the baseline.
</success_metrics>
<non_goals>
- Do not change the fixture, public API, or evaluator to improve the result.
</non_goals>
<completion_criteria>
- Median runtime on the fixed fixture is at least 15 percent below the recorded baseline.
- Parsed output is identical to the baseline output.
- Peak memory is at or below the measured baseline.
- Commands, code and fixture identities, measurements, and the final decision are recorded.
- current_state.md records accepted work and any remaining limits.
</completion_criteria>
```

- The context files must hold the real commands, fixture, and baseline.

- Success metrics track progress. Completion criteria define done, with one line per gate.

## Review Before Execution

Check the bundle before the first long run.

- Confirm the baseline can be reproduced and that the report names the fixed inputs and environment.

- Make each phase name its inputs, command, output file, gate, and failure branch.

- Check that forbidden shortcuts and edit boundaries are explicit.

- For expensive commands, verify the runner's status, checkpoint, and resume behavior before a full run.

- If you hand the goal to Codex `/goal`, run the validator and check the goal's meaning as well as its length.

```bash
python3 /absolute/path/to/objectives/.codex/skills/setup-objective/scripts/validate_goal_length.py \
  objectives/parser-benchmark/goal.md
```

## Start and Resume

Tell the agent to read the objective's goal and current state, verify the recorded artifacts, and run the next phase whose prerequisites are met.

- Use your host's goal mode if it has one. Plain agent instructions can point to the same files.

- Before resuming, check for an active process and its status record.

  - If the process is healthy, monitor it through the runner's status command.

  - If it ended, compare the checkpoint's input and config identity with the intended run, then run the documented resume command.

  - Do not start a duplicate job because the earlier conversation ended.

- A useful handoff names the last verified result, where its evidence lives, whether a process is active, and the single next action.

  - "Continue optimization" is not actionable.

  - "Inspect the completed benchmark comparison and reject changes that fail output equivalence" is actionable when it names the artifact path.

## Finish or Record a Blocker

Compare the result against every completion criterion.

- If the 15 percent target was missed, report the measured result and the failed gate.

- Keep useful rejected experiments in the objective's workspace.

- If a dependency blocks progress, record what is missing and the smallest action that would unblock it.

- Update current_state.md before the session ends.

  - If a host goal is active, its status must match the same outcome.

- Do not mark an improvement objective complete because a search reached its own stop condition.

## Adapt the Pattern to Research

For a parameter search, replace the benchmark phases with a research loop.

- The phases are baseline reproduction, a candidate matrix, cheap screening, full validation of selected candidates, and a promotion decision.

- Keep evaluation labels out of deployable rules.

- Link every accepted or rejected candidate to its config and validation artifacts.

- Write stopping criteria that can record two honest outcomes without redefining success.

  - A search can reject every candidate.

  - A stage can pass its own gate while the downstream result still needs work.
