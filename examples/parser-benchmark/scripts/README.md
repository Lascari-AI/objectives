# Scripts

Objective-local benchmark code goes here. This example ships no real code.

- `bench_parser.py`: runs warmup and timed passes on a fixture, then one peak memory pass.
    - Writes `run_manifest.json`, `timings.csv`, `summary.json`, `status.json`, and `run.log.jsonl` into `--run-dir`.
    - `--status` reads `status.json` without starting work.
- `compare_output.py`: diffs a run's parsed output against `artifacts/baseline/parsed_output.jsonl` and writes the diff count into the run's `summary.json`.
- `profile_parser.py`: runs one profiled pass and writes the sorted profile to `artifacts/profiles/`.
- Keep these scripts here, not in the repo's shared `scripts/` folder.
    - If one becomes part of the product, move it and record the move in `current_state.md`.
