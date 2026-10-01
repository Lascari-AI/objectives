Objectives are a lightweight framework that keeps a coding agent on track during long-running tasks. All you need is a prompt and a file structure, with no service, no scheduler, and nothing to deploy. The setup-objective skill writes the files, and the agent rereads them after every compaction.

The pattern came out of an image-grouping coding challenge, where a plain goal-mode run beat a custom multi-agent research harness with far less setup.

## The Constraint

Goal mode keeps one static goal prompt in context through every compaction.

- The goal prompt is capped under 4,000 characters.

- Everything else gets summarized at each compaction.

  - Run details, results, and decisions lose detail every time.

  - On a run that lasts days, you cannot trust the summary to keep what matters.

- A long run needs far more context than 4,000 characters can hold.

  - It needs explicit stopping criteria.

  - It needs to know how the work should be done, such as running experiments with MLX.

  - It needs a record of every experiment.

  - It needs to know where the run is right now.

## The Goal Prompt

The goal prompt stays short and points to files. Every objective's `goal.md` has the same six blocks.

```xml
<goal>
- What this objective must achieve.
</goal>

<context_refresh>
- Reread objectives/<slug>/goal.md.
- Reread objectives/<slug>/current_state.md.
- Reread the relevant objectives/<slug>/context/*.md files.
</context_refresh>

<working_strategy>
- The approach and the order of work.
</working_strategy>

<success_metrics>
- Observable signs of progress.
</success_metrics>

<non_goals>
- What this objective must not expand into.
</non_goals>

<completion_criteria>
- What must be true before the objective is done.
</completion_criteria>
```

- The `<context_refresh>` block does the main work.

  - It tells the agent to reread the objective's files at the start and after every compaction.

  - The files carry the context that does not fit in the goal.

- The other blocks state the outcome, the approach, the progress signals, the scope limits, and when the objective is done.

## Where Each Need Lives

Each need of a long run has its own home in the objective.

- **Stopping criteria live in goal.md.**

  - Name every metric on every dataset so the agent knows exactly when it is done.

  - "Get the accuracy higher" lets the agent stop after one dataset improves.

- **How the agent should work lives in context/.**

  - The context files hold the plan, the constraints, and project how-tos, such as running experiments with MLX.

  - They have no length limit.

- **Every experiment lives in the objective's workspace.**

  - Scripts, reports, and run outputs stay inside `objectives/<slug>/`.

  - The repo stays clean, and every file traces back to the objective that made it.

- **The current state lives in current_state.md.**

  - It records what is proven, what is running, and what comes next.

  - A restarted agent resumes from it, and you read the same file to check on a run.

```
objectives/
└── example/
    ├── artifacts/  # reports and run outputs
    ├── context/  # how the agent should work
    ├── scripts/  # experiment scripts and runners
    ├── current_state.md  # where the run is right now
    └── goal.md  # stopping criteria and the context_refresh list
```

## Pros and Cons

Objectives are the fast way to get one long task running. They do not replace a full orchestration system.

- **Pros**

  - Setup takes a prompt and a few files, with no service to build.

  - The format is built for long runs. The agent stays on track across compactions and restarts.

  - The pattern is general. It fits most tasks you can describe with clear stopping criteria.

- **Cons**

  - The run is not optimized. You cannot tune individual agents to cut cost or improve efficiency.

  - One agent follows one path. There is no fan-out to parallel candidates or separate research agents.

A fully built orchestration system would likely do better, but it takes far more time to build and tune.

## Read Next

The other pages cover setup, the file format, and resuming a run.

- Create and Resume an Objective walks through setup, a worked example, and the first handoff.

- Objective Bundle and Lifecycle defines each file, the phase contract, and completion.

- Setup Objective Skill describes what the authoring agent reads, writes, and checks.

- Scaffold and Validator documents the two Python helpers and their limits.

- The skill source is .codex/skills/setup-objective/SKILL.md.
