An objective separates stable intent from changing state. The goal and context files define valid work. `current_state.md` records the latest evidence and the next action.

## Structure

The scaffold creates eight Markdown files and, by default, an `examples/` directory.

```
objectives/
└── example/
    ├── artifacts/  # reports and run outputs, added when needed
    ├── context/
    │   ├── 00_problem.md  # baseline, failure modes, prior evidence
    │   ├── 01_constraints.md  # validity rules and forbidden shortcuts
    │   ├── 02_implementation_scope.md  # owned files and read-only boundaries
    │   ├── 03_working_plan.md  # phases, outputs, gates, recovery
    │   └── 04_validation_and_handoff.md  # acceptance and evidence requirements
    ├── examples/  # optional concrete inputs or examples
    ├── scripts/  # experiment scripts and runners, added when needed
    ├── README.md  # entry point and context map
    ├── current_state.md  # current state and handoff
    └── goal.md  # outcome, context_refresh list, and stopping criteria
```

- The objective folder is the run's workspace.

  - Scripts go in `scripts/` and reports and run outputs go in `artifacts/`, both under the objective, never at the repo root.

  - Long runs write one directory per run under `artifacts/runs/<run_id>/`.

- The scaffold does not create `scripts/` or `artifacts/`. The agent adds them when the objective needs them.

## The Goal

`goal.md` uses six pseudo-XML blocks with Markdown bullets inside them.

- The blocks are goal, context_refresh, working_strategy, success_metrics, non_goals, and completion_criteria.

  - The format is a reading convention for agents, not a parsed schema.

  - Add a block such as `<validation_commands>` only when it reduces ambiguity.

- The `<context_refresh>` block lists the files the agent rereads at the start and after every compaction or resume.

  - The scaffold lists goal.md, current_state.md, and all five context files.

- The completion criteria are the stopping criteria.

  - Name every metric on every dataset. A vague target lets the agent stop at the first easy win.

  - Success metrics describe progress. A stage can meet its metric while the final pipeline still fails validation.

- The goal stays under 4,000 characters so it fits Codex `/goal`.

  - Background, evidence, and phase plans move into `context/`.

## The Phase Contract

`context/03_working_plan.md` splits substantial work into phases. Each phase answers six questions.

| Field | Question It Answers |
| --- | --- |
| Objective | What does this phase establish? |
| Inputs | Which code, data, configs, and previous artifacts does it use? |
| Process | Which concrete actions or commands run? |
| Outputs | Which files or decisions survive the session? |
| Gate | What evidence permits the next phase? |
| Failure handling | What happens when evidence is missing or the gate fails? |

- Each phase's output becomes an input to a later phase or a final decision.

- Replace a vague task such as "validate the candidate" with the dataset, command, artifact, and acceptance condition.

## Execution and Handoff

The files carry the run across compactions and restarts.

- The agent rereads goal.md, current_state.md, and the relevant context files at the start of work and after every compaction or resume.

- The agent updates current_state.md at the start of active work, after milestones, before handoff or compaction, and before its final response.

  - The update records completed decisions, active work, next actions, risks, and important paths.

  - Link logs instead of pasting them into the current state.

- If a run is still active at handoff, current_state.md gets an `<active_runs>` entry.

  - The entry names the command, PID or session, start time, status, log, checkpoint, and control paths, and the safest next action.

## Long-Running Work

A runner that may run longer than a few minutes writes these files in its run directory.

| Artifact | Responsibility |
| --- | --- |
| run_manifest.json | Identity, command, code/input/config fingerprints, output paths, and resume command. |
| status.json and progress.jsonl | Current phase, completed units, heartbeat, and progress history. |
| run.log.jsonl | Warnings, errors, checkpoint writes, and accepted or rejected control changes. |
| checkpoint.json | Completed units and enough identity to reject an incompatible resume. |
| outputs/, partial/, rejected/ | Separate accepted evidence from incomplete or invalid output. |

- These files are a runner contract. The scaffold does not generate them.

- The runner checkpoints after bounded units of work and writes accepted state atomically.

- The runner validates code, config, and input compatibility before it resumes.

  - If a runner cannot provide a property, the working plan says why and names the nearest fallback.

- Live controls such as concurrency or stop-after-current-unit can change mid-run if the runner validates them.

  - Changes to inputs, algorithm, or validation gates start a new run segment with explicit lineage.

## Completion

Map every completion criterion to evidence before you declare the objective done.

- A failed search can end with a rejection report without meeting an improvement goal.

  - Record that outcome and the next hypothesis or blocker.

- A successful stage does not redefine the overall goal.

  - Record any approved scope change explicitly.

- Host goal status and the Markdown state are separate. Keep both describing the same outcome.

## Why

Each part of the bundle prevents one failure of long runs.

- Stable intent stops each session from inventing a new definition of success.

- A compact current state makes continuation cheap.

- Durable artifacts let a reviewer check a decision after the conversation that produced it is gone.
