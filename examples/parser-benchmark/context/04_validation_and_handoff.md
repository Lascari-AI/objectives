<validation_and_handoff>
    <validation_ladder>
        - `pytest tests/parser`: all tests pass with no new skips. Run before any benchmark.
        - `compare_output.py` on `configs_small`: zero differences. Cheap first check for output drift.
        - `bench_parser.py` on `configs_large`: median, spread, and peak memory against the baseline.
        - `compare_output.py` on `configs_large`: zero differences.
        - Two clean combined runs: both clear every gate before the objective is complete.
    </validation_ladder>

    <artifact_contract>
        - `artifacts/runs/<run_id>/run_manifest.json`: `run_id`, `command`, `revision`, `python_version`, `fixture_sha256`, `warmup`, `repeat`.
        - `artifacts/runs/<run_id>/timings.csv`: `pass_index`, `phase`, `seconds`.
        - `artifacts/runs/<run_id>/summary.json`: `median_s`, `iqr_s`, `peak_rss_mb`, `delta_vs_baseline_pct`, `output_diff_count`, `decision`, `decision_reason`.
        - `artifacts/runs/<run_id>/status.json` and `run.log.jsonl`: progress and per-pass events.
        - `artifacts/candidates.md`: one row per candidate with its final decision.
    </artifact_contract>

    <acceptance_gates>
        - Median runtime is at most 85 percent of the baseline median in both confirmation runs.
        - Output differences are zero on both fixtures.
        - Peak resident memory is at or below the baseline peak.
        - Parser tests pass.
    </acceptance_gates>

    <report_contract>
        - `report.md` must summarize the baseline, each candidate and its decision, the final comparison, and any remaining limits.
        - Link every number to the run folder that produced it.
    </report_contract>

    <current_state_update>
        - Update `current_state.md` after the baseline, after every candidate decision, and before handoff or compaction.
        - Name the last verified result, where its evidence lives, whether a benchmark is running, and the single next action.
        - If a benchmark is running at handoff, record its command, run id, status path, and log path in `<active_runs>`.
    </current_state_update>

    <blocked_or_failed_handoff>
        - If the 15 percent target is missed, report the best verified gain and the failed gate. Do not mark the objective complete.
        - Keep rejected run folders. They show what was tried and why it failed.
    </blocked_or_failed_handoff>
</validation_and_handoff>
