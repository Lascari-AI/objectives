# Objectives

This repo is a local Codex objectives workspace. It holds the skills and
references used to turn long-running agent work into something observable,
steerable, and recoverable instead of a one-shot terminal run.

The main skill is
[setup-objective](.codex/skills/setup-objective/SKILL.md). It defines the
objective bundle format, the handoff contract, and the defaults for scripts or
analysis jobs that may run for many minutes or hours.

## Why This Exists

Long-running agent work needs more than a prompt and a command. When an agent is
running a 30-minute data analysis script, a multi-hour sweep, or a 12-hour batch
job, the next agent needs to know:

- what process is running;
- what phase it is in;
- how much work is complete;
- whether progress is still moving or the run is probably hanging;
- where the logs, status, checkpoints, and partial outputs live;
- how to stop, steer, resume, or recover the run without losing completed work.

The goal is steerability. Agents will drift or inherit stale context, so the
repo should leave durable state that lets the next agent inspect the active
work, point it back at the objective, and continue from the last safe checkpoint.

## Core Contract

For long-running scripts, runners, sweeps, audits, migrations, report builders,
and data-processing jobs, default to this contract:

- **Observable:** write status, progress, logs, heartbeats, phase names,
  completed/total units, throughput, and ETA when defensible.
- **Inspectable:** record the command, PID or session, start time, status path,
  log path, checkpoint path, control/config path, and safest next action.
- **Recoverable:** checkpoint after bounded units of work and make reruns skip
  completed durable work.
- **Resumable:** provide explicit resume behavior and a dry-run resume plan when
  useful.
- **Controllable:** allow safe runtime changes such as pause, stop after current
  unit, concurrency, chunk size, rate limit, or log level when practical.
- **Handoffable:** keep `current_state.md` updated so another agent does not
  need terminal scrollback to understand what is happening.

## Skill Pieces

- [Setup Objective skill](.codex/skills/setup-objective/SKILL.md): the main
  workflow and defaults for objective bundles, long-running stability, goal
  length, and pseudo-XML goal format.
- [Long-running handoff and scripts](.codex/skills/setup-objective/references/long_running_handoff_and_scripts.md):
  the detailed contract for run directories, `status.json`, `progress.jsonl`,
  `run.log.jsonl`, `checkpoint.json`, `control.json`, script flags, interruption
  handling, and active-run handoff.
- [Objective authoring](.codex/skills/setup-objective/references/objective_authoring.md):
  how to write `goal.md`, `current_state.md`, and `context/*.md` so a fresh
  agent can execute the work without guessing.
- [Debugging and research acceleration](.codex/skills/setup-objective/references/debugging_analysis_acceleration.md):
  optional guidance for speeding up local analysis and research scripts without
  turning workstation-specific acceleration into a production dependency.
- [Objective scaffold script](.codex/skills/setup-objective/scripts/create_objective.py):
  creates a new objective bundle skeleton.
- [Goal length validator](.codex/skills/setup-objective/scripts/validate_goal_length.py):
  checks that a `goal.md` intended for Codex `/goal` fits the 4,000-character
  limit.
- [Codex config](.codex/config.toml): local feature and agent settings for this
  workspace.

## Expected Objective Shape

The skill expects objective work to be organized around durable files:

```text
objectives/<objective-slug>/
+-- README.md
+-- goal.md
+-- current_state.md
+-- context/
|   +-- 00_problem.md
|   +-- 01_constraints.md
|   +-- 02_implementation_scope.md
|   +-- 03_working_plan.md
|   +-- 04_validation_and_handoff.md
+-- examples/
+-- artifacts/
    +-- runs/<run_id>/
        +-- run_manifest.json
        +-- status.json
        +-- progress.jsonl
        +-- run.log.jsonl
        +-- checkpoint.json
        +-- control.json
        +-- outputs/
        +-- partial/
        +-- rejected/
```

For a short task, keep the structure lightweight. For a long task, especially a
data-analysis process that can run for 30 minutes or more, the status/log/
checkpoint/control files are the difference between steerable work and lost
work.

## Operating Rule

If a script would be painful to restart from zero, design it so a future agent
can answer these questions cheaply:

1. Is it still running?
2. What is it doing right now?
3. How far through the work is it?
4. Is the heartbeat fresh?
5. What has already been saved?
6. What can be safely changed while it runs?
7. What exact command resumes it after a crash?
