The skill's two Python helpers create an objective bundle and check goal length. They use only the Python standard library. They do not run the objective, schedule work, or manage checkpoints.

Both helpers support the Objective Bundle and Lifecycle contract, and their source is under `.codex/skills/setup-objective/scripts/`.

## Scaffold

.codex/skills/setup-objective/scripts/create_objective.py writes the README, goal, current state, and five context templates.

- It normalizes the name to a lowercase hyphenated slug.

- It creates `examples/.gitkeep` unless you pass `--no-examples`.

| Argument | Behavior |
| --- | --- |
| name | Required objective name or slug. |
| --title TEXT | Readable title; otherwise derived from the slug. |
| --root PATH | Output root; defaults to objectives relative to the current directory. |
| --no-examples | Skip examples/.gitkeep. |
| --date TEXT | Initial state date; defaults to today's date. Use YYYY-MM-DD. |
| --force | Overwrite existing scaffold files. This replaces content, so inspect the destination first. |

- The overwrite check runs per file, not across the whole bundle.

  - The script can write earlier files before a later existing file stops it.

  - After a failure, inspect the directory instead of rerunning with `--force`.

- The templates are deliberately incomplete.

  - A successful run means the files exist, not that the objective is ready to execute.

## Goal-Length Validator

.codex/skills/setup-objective/scripts/validate_goal_length.py checks that a goal fits under the 4,000-character Codex `/goal` limit.

- It reads a file, or stdin when no path is given.

- Before counting, it strips surrounding whitespace, an outer fenced block, and a leading `/goal` command followed by whitespace.

| Argument | Behavior |
| --- | --- |
| path | Optional file. When omitted, read stdin. |
| --max-chars N | Hard maximum; default 3,999. |
| --target-chars N | Optional advisory target; no target by default. |
| --compact-target | Use the 1,600-character advisory target unless an explicit target is set. |
| --strict-target | Fail above the selected target; default target becomes 1,600 if none was chosen. |

- It prints `objective_chars`, `target_chars`, and `max_chars`.

- It exits with code 1 above the hard maximum of 3,999 characters, or above a target enforced with `--strict-target`.

  - An advisory target warning alone still exits with code 0.

- An empty goal passes the length check, so review the goal's meaning as well.

- The default maximum is the Codex TUI limit recorded by the skill. Other clients and hosts may set different limits.

## Implementation Choices

The helpers stay small because the agent and its runners do the work.

- Markdown keeps the objective readable with ordinary repository tools.

- Pseudo-XML marks sections without a runtime parser.

- Each project runner implements its own compatibility checks, atomic checkpoints, and controls.

  - A documented flag such as `--resume` does not prove that an older runner meets the current guidance.

## Skill Metadata

.codex/skills/setup-objective/agents/openai.yaml holds the Codex display name and default skill prompt.

- The repository's `.codex/config.toml` is local Codex configuration, not an install requirement.

- Copying the skill does not require copying that file.
