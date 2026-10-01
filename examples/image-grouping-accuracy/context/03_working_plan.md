<working_plan>
    <overview>
        1. baseline_reproduction - Score the current pipeline on every shard of all four datasets and write the baseline gate table.
        2. candidate_matrix_design - Build a config matrix from failure cohorts and feature hypotheses.
        3. proxy_sweep - Screen the matrix on dev shards with cached features, using MLX on the local GPU.
        4. visual_review - Render failures for top candidates and have a sub-agent turn them into feature hypotheses.
        5. full_validation - Run finalists through the real pipeline on dev, then holdout, for all four datasets.
        6. promotion_and_handoff - Promote one config only if all 12 gates and the audits pass.
    </overview>

    <operating_principles>
        - All four datasets, every round. A candidate scored on one dataset has no decision yet.
        - Rank by the weakest gate, not by mean. The gate with the smallest margin decides which candidates move on.
        - False merges cost more than false splits. When a candidate trades one for the other, require a brake.
        - Research speed is welcome. Production portability is required.
        - Phases 3 to 5 loop until all gates pass or the search is exhausted.
    </operating_principles>

    <script_entrypoints>
        - `proxy_sweep_mlx.py --matrix <csv> --run-dir <dir>`: starts a sweep and writes all files under `<dir>`.
        - `proxy_sweep_mlx.py --status --run-dir <dir>`: reads `status.json` without starting work.
        - `proxy_sweep_mlx.py --resume --run-dir <dir>`: checks the checkpoint against the matrix and cache fingerprints, then continues.
        - `<dir>/control.json`: safe live controls, read every 30 seconds.
    </script_entrypoints>

    <runtime_observability>
        - `status.json`: phase, configs done and total, current dataset and shard, configs per minute, ETA, last heartbeat, and backend (`mlx` or `numpy`).
        - `progress.jsonl`: one event per finished config block.
        - `run.log.jsonl`: phase changes, checkpoint writes, control changes, warnings, and errors.
        - Heartbeat every 30 seconds. A heartbeat older than 5 minutes means the run is probably hung.
    </runtime_observability>

    <checkpoint_resume>
        - `checkpoint.json`: written atomically after every block of 512 configs.
            - Holds the matrix fingerprint, feature cache fingerprints, code revision, and finished config ids.
        - Resume skips finished configs. If any fingerprint changed, refuse to resume and start a new run id.
    </checkpoint_resume>

    <runtime_controls>
        - Mutable: `config_block_size`, `pause`, `stop_after_current_block`, `log_level`.
        - Immutable: the config matrix, feature definitions, shard lists, and backend. Changing any of these starts a new run.
    </runtime_controls>

    <phase id="1" name="baseline_reproduction">
        <objective>
            - Record the current pipeline's per-shard scores and gate table for all four datasets.
        </objective>
        <inputs>
            - `pipeline/grouping/config/default.json` at the current revision.
            - Dev and holdout shard lists for random10, random20, random40, and random80.
        </inputs>
        <process>
            - Run `eval/run_shards.py` for each dataset on dev, then on holdout.
            - Run `gate_table.py` on the results.
            - Split each shard's errors into false merges, false splits, and chain merges.
            - Build the feature cache for all dev shards.
        </process>
        <outputs>
            - `artifacts/baseline/per_shard_scores.csv`: `dataset`, `shard_id`, `split`, `score`, `exact`, `truth_groups`, `false_merges`, `false_splits`, `chain_merges`.
            - `artifacts/baseline/gate_table.csv`: `dataset`, `metric`, `value`, `gate`, `margin`, `pass`.
        </outputs>
        <gate>
            - Every shard of every dataset has a score, and the gate table has all 12 rows.
        </gate>
        <failure_handling>
            - If a shard fails to run, record it in `artifacts/baseline/missing_shards.csv` and fix the run before tuning. A missing shard makes the worst gate meaningless.
        </failure_handling>
    </phase>

    <phase id="2" name="candidate_matrix_design">
        <objective>
            - Build a config matrix that targets the failing gates, with brakes for every merge-raising change.
        </objective>
        <inputs>
            - Baseline gate table and per-shard errors.
            - The latest `artifacts/visual_review/<round_id>/synthesis.csv`, if one exists.
        </inputs>
        <process>
            - Name the failing gates and the error type that drives each one.
            - For each feature hypothesis, add a baseline row, conservative and loose threshold bands, a one-factor ablation, and a version with and without its brake.
            - Mark any row that needs truth labels or metadata as `diagnostic_only`. It may not be promoted.
        </process>
        <outputs>
            - `artifacts/config_matrix.csv`: `config_id`, `family`, `hypothesis_id`, `feature_sources`, thresholds, `brake`, `ablation_of`, `diagnostic_only`, `reason`.
        </outputs>
        <gate>
            - Every failing gate has at least one family aimed at it, and every merge-raising row has a braked twin.
        </gate>
        <failure_handling>
            - If a hypothesis needs a feature the cache lacks, add it to `features.py`, rebuild the cache, and record the new cache fingerprint.
        </failure_handling>
    </phase>

    <phase id="3" name="proxy_sweep">
        <objective>
            - Screen the whole matrix on dev shards cheaply and pick finalists.
        </objective>
        <inputs>
            - `artifacts/config_matrix.csv` and the dev feature cache.
        </inputs>
        <process>
            - Before the first sweep, run a parity check: score 50 configs on two shards with `--backend mlx` and `--backend numpy`. Results must match.
            - Run the sweep with MLX. Keep large feature arrays on the GPU and copy back only per-shard counts.
            - Compute proxy gate margins per dataset for every config.
            - Select finalists: the top 3 by weakest-gate margin, plus 2 near-misses that fail one gate by the smallest amount.
        </process>
        <outputs>
            - `artifacts/runs/<run_id>/outputs/proxy_results.csv`: `config_id`, `dataset`, `shard_id`, `proxy_score`, `false_merges`, `false_splits`.
            - `artifacts/runs/<run_id>/outputs/finalists.csv`: `config_id`, `weakest_gate`, `margin`, `selection_reason`.
        </outputs>
        <gate>
            - Parity check passes, and at least one finalist improves the weakest baseline gate on dev.
        </gate>
        <failure_handling>
            - If MLX and NumPy disagree, stop using MLX for this sweep, run it with `--backend numpy`, and record the mismatch in state.
            - If no config beats the baseline's weakest gate, go to phase 4 with baseline failures to find new hypotheses.
        </failure_handling>
    </phase>

    <phase id="4" name="visual_review">
        <objective>
            - Turn failures that numbers cannot explain into feature hypotheses for the next matrix.
        </objective>
        <inputs>
            - Finalists from phase 3 and their per-shard errors.
        </inputs>
        <process>
            - Render sheets with `render_failures.py`: new false merges, new false splits, fixed cases, and same-scene controls near risky merges.
            - Give the sheets to a sub-agent with an observation-only contract.
                - It reports whether each case looks safe, unsafe, or ambiguous, and names the visible pattern, such as exposure bracket, clipping, same-room lookalike, or viewpoint shift.
                - It suggests image-content features that could separate the case.
                - It must not edit code, tune thresholds, or use filenames, shard ids, or truth labels in a suggestion.
            - Merge the reports into one synthesis. Map each recurring pattern to a candidate feature, a brake feature, and the next sweep family.
        </process>
        <outputs>
            - `artifacts/visual_review/<round_id>/sheets/`: rendered sheets with an index CSV.
            - `artifacts/visual_review/<round_id>/reports/`: raw sub-agent reports.
            - `artifacts/visual_review/<round_id>/synthesis.csv`: `case_id`, `config_id`, `change_type`, `visual_pattern`, `visual_safety`, `feature_hypothesis`, `brake_hypothesis`, `next_sweep_family`.
        </outputs>
        <gate>
            - Each recurring pattern has a testable hypothesis built from image content, and phase 2 has new matrix rows for it.
        </gate>
        <failure_handling>
            - If a report suggests a rule based on names, order, or a specific image, drop that suggestion and note why in the synthesis.
        </failure_handling>
    </phase>

    <phase id="5" name="full_validation">
        <objective>
            - Confirm finalists with the real pipeline on all four datasets.
        </objective>
        <inputs>
            - `finalists.csv` from phase 3.
        </inputs>
        <process>
            - Run `eval/run_shards.py` on dev for each finalist and each dataset, then build its gate table.
            - Score holdout only for finalists that pass every dev gate.
            - Apply the risk budget in `01_constraints.md`. Reject any finalist that lowers a worst shard, adds too many false merges, or shows a holdout gap.
        </process>
        <outputs>
            - `artifacts/runs/<run_id>/outputs/per_shard_scores.csv` for each finalist.
            - `artifacts/gate_table.csv`: one block of 12 rows per finalist, with `split` and `pass`.
        </outputs>
        <gate>
            - One finalist passes all 12 gates on dev and holdout.
        </gate>
        <failure_handling>
            - If no finalist passes, record the gates each one missed and loop back to phase 2.
        </failure_handling>
    </phase>

    <phase id="6" name="promotion_and_handoff">
        <objective>
            - Promote the passing config into production and leave a verifiable record.
        </objective>
        <inputs>
            - The passing finalist and its gate table.
        </inputs>
        <process>
            - Write its thresholds into `pipeline/grouping/config/default.json`.
            - Run `audit_leakage.py`, `pytest tests/grouping` in an environment without MLX, and one CPU run per dataset to confirm the same groups.
            - Write `report.md` and update `current_state.md`.
        </process>
        <outputs>
            - `objectives/image-grouping-accuracy/report.md`: baseline, accepted changes, rejected families, final gate table, and limits.
            - `artifacts/promotion/cpu_parity.csv`: `dataset`, `shard_id`, `groups_match`.
        </outputs>
        <gate>
            - Every completion criterion in `goal.md` is checked against a named artifact.
        </gate>
        <failure_handling>
            - If the CPU run differs from the research result, do not promote. Find the cause and record it.
        </failure_handling>
    </phase>
</working_plan>
