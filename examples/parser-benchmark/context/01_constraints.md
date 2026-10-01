<constraints>
    <hard_rules>
        - Compare every experiment to the recorded baseline, not to the previous experiment.
        - Use the same fixture fingerprint, interpreter version, warmup count, and repeat count as the baseline.
        - Change one mechanism per experiment so each result has a clear cause.
        - Keep the public API in `src/parser/__init__.py` unchanged.
    </hard_rules>

    <forbidden_shortcuts>
        - Editing or trimming the fixture is invalid because it changes what is measured.
        - Changing the benchmark or comparison scripts after the baseline is invalid unless the baseline is rerun with the new scripts.
        - Caching parsed results across benchmark passes is invalid because it measures the cache, not the parser.
        - Skipping validation work, such as duplicate key checks, is invalid because it changes behavior.
        - Adding a dependency or native extension is out of scope.
    </forbidden_shortcuts>

    <data_and_feature_boundaries>
        - `fixtures/bench/configs_large/` is the only fixture used for the runtime and memory gates.
        - `fixtures/bench/configs_small/` is a sanity fixture for output equivalence only. Do not tune against it.
    </data_and_feature_boundaries>

    <risk_budget>
        - Run-to-run spread: if the interquartile range of a run is above 3 percent of its median, rerun before deciding.
        - Peak memory: zero budget. Any increase over the baseline peak rejects the experiment.
    </risk_budget>

    <promotion_or_completion_gates>
        - `runtime_gate`: candidate median is at most 85 percent of the baseline median.
        - `output_gate`: `compare_output.py` reports zero differences on both fixtures.
        - `memory_gate`: candidate peak resident memory is at or below the baseline peak.
        - `test_gate`: `pytest tests/parser` passes with no new skips.
    </promotion_or_completion_gates>
</constraints>
