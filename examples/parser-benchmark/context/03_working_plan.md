<working_plan>
    <overview>
        1. baseline_reproduction - Measure the current parser on the fixed fixture and save its output.
        2. bottleneck_profile - Find where the time goes and rank candidate mechanisms.
        3. single_mechanism_experiments - Change one mechanism at a time and measure it against the baseline.
        4. combination_and_confirmation - Combine accepted changes and confirm all gates with a fresh run.
        5. decision_and_handoff - Record the final decision and update state.
    </overview>

    <operating_principles>
        - Correctness first. A faster run with any output difference is rejected, not fixed later.
        - One mechanism per experiment, so every accepted change has a measured cause.
        - Decide from medians over repeated runs, never from a single pass.
        - When speed and memory conflict, memory wins. The gate has no budget.
    </operating_principles>

    <runtime_observability>
        - Each benchmark run takes about 4 minutes, so it gets a light version of the long-running contract.
        - `artifacts/runs/<run_id>/status.json`: phase (`warmup`, `timed`, `memory`, `done`), completed passes, total passes, last heartbeat, and output paths.
        - `artifacts/runs/<run_id>/run.log.jsonl`: one event per pass with its duration, plus start, end, and errors.
    </runtime_observability>

    <checkpoint_resume>
        - No mid-run checkpoint. A 4 minute run is cheaper to repeat than to resume.
        - If a run is interrupted, mark its status `interrupted`, keep the folder for reference, and start a new run id.
        - Never merge passes from two runs into one median.
    </checkpoint_resume>

    <runtime_controls>
        - No live controls. Warmup count, repeat count, fixture, and interpreter are fixed for every run because they change what the median means.
    </runtime_controls>

    <phase id="1" name="baseline_reproduction">
        <objective>
            - Record the baseline median, spread, peak memory, and canonical parsed output.
        </objective>
        <inputs>
            - Parser source at the current main revision.
            - `fixtures/bench/configs_large/` and `fixtures/bench/configs_small/`.
        </inputs>
        <process>
            - Write `bench_parser.py`, `compare_output.py`, and `profile_parser.py` in `objectives/parser-benchmark/scripts/`.
            - Run the benchmark command twice with run ids `baseline-a` and `baseline-b`.
            - If the two medians differ by more than 3 percent, close other heavy processes and rerun both. Otherwise, use `baseline-a` as the baseline.
            - Save parsed output for both fixtures.
        </process>
        <outputs>
            - `artifacts/baseline/baseline_summary.json`: `revision`, `python_version`, `fixture_sha256`, `warmup`, `repeat`, `median_s`, `iqr_s`, `peak_rss_mb`, `command`.
            - `artifacts/baseline/parsed_output.jsonl`: one line per file with path and parsed value or error.
        </outputs>
        <gate>
            - Two baseline runs agree within 3 percent and the summary has every field filled.
        </gate>
        <failure_handling>
            - If runs never agree within 3 percent, record the spread in `current_state.md` and raise the repeat count for all runs before continuing.
        </failure_handling>
    </phase>

    <phase id="2" name="bottleneck_profile">
        <objective>
            - Turn the profile into a ranked list of mechanisms to try.
        </objective>
        <inputs>
            - Baseline revision.
            - `artifacts/baseline/baseline_summary.json`.
        </inputs>
        <process>
            - Run `profile_parser.py` for one pass and save the output.
            - List the top 10 functions by cumulative time with their share of the pass.
            - For each hot function, name one mechanism that could make it cheaper, such as fewer string copies, a precompiled lookup, or an early type check.
            - Rank mechanisms by expected gain and by memory risk.
        </process>
        <outputs>
            - `artifacts/profiles/baseline_profile.txt`: raw profile.
            - `artifacts/candidates.md`: table with `candidate_id`, `target_function`, `mechanism`, `expected_gain`, `memory_risk`, `status`.
        </outputs>
        <gate>
            - At least three candidates exist, and together their target functions cover at least 40 percent of pass time.
        </gate>
        <failure_handling>
            - If no function takes more than 10 percent of time, record a flat profile in state. The 15 percent target may need several small wins, so plan for phase 4 combinations.
        </failure_handling>
    </phase>

    <phase id="3" name="single_mechanism_experiments">
        <objective>
            - Measure each candidate alone and accept or reject it on all four gates.
        </objective>
        <inputs>
            - `artifacts/candidates.md`.
            - Baseline summary and parsed output.
        </inputs>
        <process>
            - Take the highest ranked untested candidate and implement it on a branch named `parser-bench/<candidate_id>`.
            - Run `pytest tests/parser`. If it fails, reject the candidate and record the failing test.
            - Run the benchmark with run id `<candidate_id>-r1`, then run `compare_output.py`.
            - If output differs or peak memory rises, reject. If the median gain is under 2 percent, mark it `no_effect`. Otherwise, accept it for phase 4.
            - Update the candidate row and `current_state.md` after every decision.
        </process>
        <outputs>
            - `artifacts/runs/<candidate_id>-r1/`: full run folder.
            - `artifacts/candidates.md`: updated `status` and `decision_reason`.
        </outputs>
        <gate>
            - Every ranked candidate has a decision, or accepted candidates already reach a combined estimate of 15 percent.
        </gate>
        <failure_handling>
            - If every candidate is rejected or has no effect, go back to phase 2 with a line-level profile of the top two functions.
            - If two rounds of phase 2 still find nothing, stop and report the best verified gain against the target.
        </failure_handling>
    </phase>

    <phase id="4" name="combination_and_confirmation">
        <objective>
            - Combine accepted mechanisms and confirm the gates on a clean run.
        </objective>
        <inputs>
            - Accepted candidates from phase 3.
        </inputs>
        <process>
            - Apply accepted changes together on `parser-bench/combined`.
            - Run tests, two benchmark runs (`combined-r1`, `combined-r2`), and output comparison.
            - If the combined gain is lower than the best single gain, remove changes one at a time to find the conflict.
        </process>
        <outputs>
            - `artifacts/runs/combined-r1/` and `artifacts/runs/combined-r2/`.
            - `artifacts/final_comparison.md`: baseline versus combined for median, spread, peak memory, output diff count, and tests.
        </outputs>
        <gate>
            - Both combined runs clear every gate in `01_constraints.md`.
        </gate>
        <failure_handling>
            - If the combined runs clear output and memory but miss 15 percent, record the measured gain and return to phase 2 for more candidates.
        </failure_handling>
    </phase>

    <phase id="5" name="decision_and_handoff">
        <objective>
            - Record the outcome so another agent or a reviewer can verify it.
        </objective>
        <inputs>
            - `artifacts/final_comparison.md` and all run folders.
        </inputs>
        <process>
            - Write `report.md` with baseline, accepted and rejected candidates, final numbers, and remaining limits.
            - Update `current_state.md` with the final status and next action.
        </process>
        <outputs>
            - `objectives/parser-benchmark/report.md`.
            - Updated `current_state.md`.
        </outputs>
        <gate>
            - Every completion criterion in `goal.md` is checked against a named artifact.
        </gate>
        <failure_handling>
            - If any criterion is not met, mark the objective as not complete and name the failed gate.
        </failure_handling>
    </phase>
</working_plan>
