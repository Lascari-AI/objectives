<implementation_scope>
    <owned_surfaces>
        - `src/parser/lexer.py`: token scanning. Main target for experiments.
        - `src/parser/values.py`: scalar conversion. Allowed target for experiments.
        - `src/parser/reader.py`: file reading and decoding. Allowed only if the profile shows I/O in the top five entries.
        - `objectives/parser-benchmark/scripts/`: benchmark, profiling, and comparison scripts.
        - `objectives/parser-benchmark/artifacts/`: baseline, profiles, and run outputs.
    </owned_surfaces>

    <read_only_references>
        - `src/parser/__init__.py`: public API. Read to confirm signatures, do not change.
        - `fixtures/bench/configs_large/`: benchmark fixture. Never edit.
        - `fixtures/bench/configs_small/`: sanity fixture. Never edit.
        - `tests/parser/`: existing behavior tests. Add tests only for new internal helpers.
    </read_only_references>

    <generated_outputs>
        - `artifacts/baseline/baseline_summary.json`: baseline measurements and identities.
        - `artifacts/baseline/parsed_output.jsonl`: canonical parsed output, one line per fixture file.
        - `artifacts/profiles/<run_id>_profile.txt`: cProfile output for one pass.
        - `artifacts/runs/<run_id>/`: one folder per experiment. Contents are listed in `04_validation_and_handoff.md`.
    </generated_outputs>

    <commands_and_entrypoints>
        - `python objectives/parser-benchmark/scripts/bench_parser.py --fixture fixtures/bench/configs_large --warmup 3 --repeat 15 --run-dir objectives/parser-benchmark/artifacts/runs/<run_id>`: timed passes plus one peak memory pass. Takes about 4 minutes.
        - `python objectives/parser-benchmark/scripts/bench_parser.py --status --run-dir <run_dir>`: prints the current phase, completed passes, and last heartbeat.
        - `python objectives/parser-benchmark/scripts/compare_output.py --baseline objectives/parser-benchmark/artifacts/baseline/parsed_output.jsonl --run-dir <run_dir>`: diffs parsed output on both fixtures.
        - `python objectives/parser-benchmark/scripts/profile_parser.py --fixture fixtures/bench/configs_large --out objectives/parser-benchmark/artifacts/profiles/<run_id>_profile.txt`: one profiled pass.
        - `pytest tests/parser`: behavior tests.
    </commands_and_entrypoints>

    <adjacent_surfaces_requiring_caution>
        - `src/parser/errors.py`: error messages are part of parsed output for invalid files. Changes here count as output drift.
        - `src/config/loader.py`: calls the parser at service start. Read it, but leave caching decisions to its owners.
    </adjacent_surfaces_requiring_caution>

    <out_of_scope>
        - The config schema and the fixture contents.
        - Parallel parsing across files. It changes the caller's contract and is a separate objective.
        - Rewriting the parser in another language.
    </out_of_scope>
</implementation_scope>
