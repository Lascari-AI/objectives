# Long-Running Handoff And Scripts

Use this reference when an objective creates, modifies, or depends on scripts
that may run for many minutes or hours. The script should be designed as a
resumable operation with durable evidence, not as a one-shot terminal command.

The core contract is:

- observable: a future agent can tell what is running, what phase it is in,
  what it has completed, and whether it is healthy;
- durable: logs, status, checkpoints, partial outputs, and final outputs are
  written under objective-local or artifact-local paths;
- resumable: interruption does not force completed work to be repeated unless a
  semantic input changed;
- controllable: safe runtime variables can be adjusted without restart when
  practical;
- handoffable: `current_state.md` identifies active runs and the safest next
  action without requiring a human to inspect terminal scrollback.

## When To Apply

Apply this contract to objective-local runners, sweeps, audits, migrations,
batch jobs, report builders, data-processing scripts, or validation jobs when
any of these are true:

- expected runtime is longer than a few minutes;
- the job has many independent units, shards, configs, files, rows, or phases;
- rerunning from zero would waste meaningful time;
- a future agent may inherit the run after compaction, disconnect, or crash;
- concurrency, chunk size, rate limits, pause/resume, or stop conditions may
  need adjustment while the job is active.

If the job is tiny, keep the implementation simple. The minimum for short jobs
is still a clear command, output path, and failure behavior.

## Artifact Layout

Prefer one run directory per long-running invocation:

```text
objectives/<slug>/artifacts/runs/<run_id>/
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

Required responsibilities:

- `run_manifest.json`: command, argv, cwd, environment assumptions, run id,
  start time, code/input/config fingerprints, immutable variables, mutable
  controls, output paths, and resume command.
- `status.json`: cheap status snapshot for humans and agents.
- `progress.jsonl`: append-only progress events for throughput and ETA
  reconstruction.
- `run.log.jsonl`: structured logs for phase changes, warnings, errors,
  checkpoint writes, control changes, and completion state.
- `checkpoint.json`: atomic checkpoint for resume.
- `control.json`: optional runtime control file for safe live changes.
- `outputs/`: accepted final or phase-complete artifacts.
- `partial/`: resumable intermediate artifacts.
- `rejected/`: outputs that were produced but failed gates.

Use temp files plus atomic rename for files that represent the latest accepted
state, especially `status.json` and `checkpoint.json`.

## Script Entrypoints

Long-running scripts should expose explicit commands or flags for the common
operational tasks:

```bash
python objectives/<slug>/scripts/run_long_job.py \
  --run-dir objectives/<slug>/artifacts/runs/<run_id> \
  --config objectives/<slug>/context/job_config.json

python objectives/<slug>/scripts/run_long_job.py --status \
  --run-dir objectives/<slug>/artifacts/runs/<run_id>

python objectives/<slug>/scripts/run_long_job.py --resume \
  --run-dir objectives/<slug>/artifacts/runs/<run_id>

python objectives/<slug>/scripts/run_long_job.py --dry-run-resume-plan \
  --run-dir objectives/<slug>/artifacts/runs/<run_id>

python objectives/<slug>/scripts/run_long_job.py --force-rebuild \
  --run-dir objectives/<slug>/artifacts/runs/<run_id>
```

Recommended flags:

- `--run-dir`: durable location for all run artifacts.
- `--status`: print a concise human-readable summary from `status.json`.
- `--resume`: validate checkpoint compatibility and continue incomplete units.
- `--resume-from`: resume from a specific phase, shard, config, or unit when
  that is safe and well-defined.
- `--dry-run-resume-plan`: print what would be skipped, rerun, or rejected.
- `--force-rebuild`: ignore prior checkpoints and rebuild outputs.
- `--control-file`: path to runtime controls, defaulting to
  `<run-dir>/control.json`.
- `--checkpoint-interval`: bounded units or seconds between checkpoint writes.
- `--log-level`: runtime log verbosity.

## Status Contract

`status.json` should be cheap to read and safe to update frequently:

```json
{
  "run_id": "20260523-153000-main-sweep",
  "state": "running",
  "phase": "proxy_sweep",
  "started_at": "2026-05-23T15:30:00-05:00",
  "updated_at": "2026-05-23T16:05:15-05:00",
  "heartbeat_at": "2026-05-23T16:05:15-05:00",
  "elapsed_seconds": 2115,
  "completed_units": 420,
  "total_units": 1000,
  "recent_units_per_minute": 12.4,
  "eta_seconds": 2806,
  "current_unit": "config_0420",
  "active_config": "objectives/<slug>/context/job_config.json",
  "control_file": "objectives/<slug>/artifacts/runs/<run_id>/control.json",
  "checkpoint": "objectives/<slug>/artifacts/runs/<run_id>/checkpoint.json",
  "log": "objectives/<slug>/artifacts/runs/<run_id>/run.log.jsonl",
  "outputs": ["objectives/<slug>/artifacts/runs/<run_id>/outputs/results.csv"],
  "resume_command": "python objectives/<slug>/scripts/run_long_job.py --resume --run-dir ...",
  "last_error": null
}
```

Use `eta_seconds: null` when total work or throughput is too uncertain for a
defensible estimate. Do not fabricate precision.

## Log Contract

Prefer JSONL for machine-readable logs. Each line should be one event:

```json
{"ts":"2026-05-23T15:31:00-05:00","level":"info","event":"phase_start","phase":"proxy_sweep","run_id":"..."}
{"ts":"2026-05-23T15:35:00-05:00","level":"info","event":"progress","completed_units":50,"total_units":1000,"units_per_minute":13.1}
{"ts":"2026-05-23T15:36:00-05:00","level":"info","event":"checkpoint_written","checkpoint":".../checkpoint.json","completed_units":62}
{"ts":"2026-05-23T15:40:00-05:00","level":"warning","event":"control_rejected","field":"concurrency","requested":999,"reason":"above max 32"}
```

Log these event classes at minimum:

- run start and run end;
- phase start and phase end;
- progress and heartbeat;
- checkpoint written;
- output accepted, partial output written, output rejected;
- control file loaded, control accepted, control rejected;
- warnings, retries, recoverable errors, fatal errors;
- interruption received and shutdown/resume state written.

Do not rely on terminal output as the only log.

## Checkpoint And Resume

Checkpoint after bounded units of work, not only at process exit.

Checkpoint metadata should include:

- run id and script version;
- command and relevant argv;
- code fingerprint or git revision when available;
- input paths and fingerprints;
- config path and fingerprint;
- immutable variables;
- completed unit ids;
- in-progress unit ids and their recovery policy;
- partial output paths;
- accepted output paths;
- rejected output paths and reasons;
- next suggested command.

Resume behavior should:

- validate code/config/input compatibility before doing work;
- skip completed durable units;
- repair or discard incomplete temp files according to a documented rule;
- keep accepted outputs separate from partial and rejected outputs;
- report a dry-run resume plan when requested;
- refuse to resume if semantic inputs changed and the script cannot guarantee
  correctness.

When a semantic input changes, create a new run segment or require
`--force-rebuild`. Do not silently mix incompatible results.

## Runtime Controls

Where safe, support live control without stopping the run. A simple pattern is
to poll `control.json` every N seconds or every N units and apply validated
changes.

Safe mutable controls often include:

- `concurrency`;
- `chunk_size`;
- `rate_limit`;
- `pause`;
- `stop_after_current_unit`;
- `checkpoint_interval`;
- `log_level`;
- `max_units`;
- `worker_restart_after_units`.

Immutable or segment-defining values usually include:

- input dataset paths or fingerprints;
- algorithm version;
- output schema;
- validation gates;
- candidate matrix contents;
- feature definitions;
- random seed when it affects result meaning;
- production/deployability rules.

Runtime-control rules:

- validate every requested change before applying it;
- clamp or reject unsafe values instead of crashing;
- log the old value, requested value, applied value, and reason;
- expose the current accepted controls in `status.json`;
- treat semantic changes as a new run segment with explicit lineage.

## Interruption Handling

Handle `SIGINT` and `SIGTERM` intentionally when the language/runtime supports
it:

1. Stop scheduling new work.
2. Let safe in-flight units finish, or checkpoint them as incomplete.
3. Flush logs and progress files.
4. Atomically write the latest checkpoint and status.
5. Mark state as `interrupted`, `stopping`, `failed`, or `complete`.
6. Record a resume command.

If immediate termination prevents cleanup, the next `--resume` should inspect
temp files and the last checkpoint before continuing.

## Handoff Contract

Do not paste large logs into `current_state.md`. Link the durable artifacts and
summarize only decision-relevant evidence.

When a run is active during handoff, add or update an `active_runs` block:

```xml
<active_runs>
    <run id="20260523-153000-main-sweep">
        <command>
            - `python objectives/<slug>/scripts/run_long_job.py --run-dir objectives/<slug>/artifacts/runs/20260523-153000-main-sweep`
        </command>
        <pid_or_session>
            - `PID 12345` or `exec session 17` if available.
        </pid_or_session>
        <started_at>
            - `2026-05-23T15:30:00-05:00`
        </started_at>
        <status_path>
            - `objectives/<slug>/artifacts/runs/20260523-153000-main-sweep/status.json`
        </status_path>
        <log_path>
            - `objectives/<slug>/artifacts/runs/20260523-153000-main-sweep/run.log.jsonl`
        </log_path>
        <checkpoint_path>
            - `objectives/<slug>/artifacts/runs/20260523-153000-main-sweep/checkpoint.json`
        </checkpoint_path>
        <control_path>
            - `objectives/<slug>/artifacts/runs/20260523-153000-main-sweep/control.json`
        </control_path>
        <safe_next_action>
            - Check `status.json`; if heartbeat is stale by more than [limit],
              inspect the last 100 log events and run the dry-run resume plan.
        </safe_next_action>
    </run>
</active_runs>
```

For completed, failed, or blocked runs, record:

- final state;
- summary of result or blocker;
- final status/log/checkpoint paths;
- accepted outputs;
- rejected or partial outputs worth preserving;
- exact resume, retry, rollback, or cleanup command;
- smallest useful next action.

## Objective Context Hooks

In `context/03_working_plan.md`, include these blocks when relevant:

```xml
<script_entrypoints>
    - `[main command]`: starts the run and writes all artifacts under
      `[run-dir]`.
    - `[status command]`: reads `[status path]` without starting work.
    - `[resume command]`: resumes after validating checkpoint compatibility.
    - `[control mechanism]`: updates safe runtime variables.
</script_entrypoints>

<runtime_observability>
    - `[status path]`: status schema and heartbeat cadence.
    - `[log path]`: JSONL event classes and retention expectation.
    - `[progress path]`: progress event schema and ETA policy.
</runtime_observability>

<checkpoint_resume>
    - `[checkpoint path]`: checkpoint cadence, compatibility metadata, and
      atomic write behavior.
    - `[resume behavior]`: completed-work detection, temp-file recovery, and
      incompatible-change handling.
</checkpoint_resume>

<runtime_controls>
    - Mutable controls: [safe live controls].
    - Immutable controls: [variables that require new run segment or rebuild].
</runtime_controls>
```

In `context/04_validation_and_handoff.md`, include:

```xml
<active_run_handoff>
    - Before compaction or final response, update `current_state.md` with the
      active command, PID/session if available, start time, status path, log
      path, checkpoint path, control/config path, and safe next action.
</active_run_handoff>
```

## Practical Fallbacks

If a script cannot reasonably implement the full contract, state the reason and
provide the nearest practical fallback:

- no dynamic controls: write a clear stop/resume path and a safe way to change
  config between run segments;
- unknown total work: report completed units, recent throughput, current phase,
  and `eta_seconds: null`;
- no stable unit ids: checkpoint by phase boundary and preserve partial output
  manifests;
- external tool with weak logging: wrap it with a parent script that records
  command, start time, heartbeat, stdout/stderr paths, exit code, and output
  existence checks;
- no safe resume: write partial artifacts and a failure note explaining why
  rerun from zero is required.
