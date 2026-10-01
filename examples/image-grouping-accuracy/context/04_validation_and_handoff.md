<validation_and_handoff>
    <validation_ladder>
        - MLX and NumPy parity on two dev shards: identical counts. Required before any MLX sweep is trusted.
        - Proxy sweep on dev shards: screening only. Never used for completion.
        - Real pipeline on dev shards, all four datasets: finalist gate table.
        - Real pipeline on holdout shards, all four datasets: only for finalists that pass every dev gate.
        - Leakage audit, CPU tests, and CPU parity: required for promotion.
    </validation_ladder>

    <artifact_contract>
        - `artifacts/gate_table.csv`: `config_id`, `split`, `dataset`, `metric`, `value`, `gate`, `margin`, `pass`.
        - `artifacts/runs/<run_id>/`: `run_manifest.json`, `status.json`, `progress.jsonl`, `run.log.jsonl`, `checkpoint.json`, `control.json`, and `outputs/`.
        - `artifacts/config_matrix.csv`: every candidate with its family, hypothesis, and decision.
        - `artifacts/visual_review/<round_id>/synthesis.csv`: visual patterns mapped to feature hypotheses.
        - `artifacts/promotion/`: leakage audit output, test log, and CPU parity table.
    </artifact_contract>

    <acceptance_gates>
        - All 12 gates in `goal.md` pass for one config on the full shard set of each dataset.
        - Leakage audit is clean.
        - CPU tests pass without MLX installed.
        - CPU groups match the research result on every shard.
    </acceptance_gates>

    <report_contract>
        - `report.md` must show the baseline and final gate tables side by side.
        - It must list accepted changes, rejected families with reasons, and visual review rounds with their outcomes.
        - Link every number to the run folder that produced it.
    </report_contract>

    <current_state_update>
        - Update `current_state.md` after the baseline, after each sweep, after each visual review round, and before handoff or compaction.
        - Always include the current gate table for the best finalist, with each gate marked pass or fail.
        - If a sweep is running, record its command, PID, start time, status path, log path, checkpoint path, control path, and safe next action in `<active_runs>`.
    </current_state_update>

    <active_run_handoff>
        - Before compaction or the final response, check every active sweep's `status.json`.
        - If the heartbeat is fresh, leave it running and record it. If it is stale, run `--dry-run-resume-plan` and record the result.
        - Never start a second sweep on the same matrix because the earlier conversation ended.
    </active_run_handoff>

    <blocked_or_failed_handoff>
        - If the search is exhausted with gates still failing, report the best config's gate table and the gates it misses. Do not mark the objective complete.
        - Name the smallest next step, such as a new feature family or a data issue for a separate objective.
    </blocked_or_failed_handoff>
</validation_and_handoff>
