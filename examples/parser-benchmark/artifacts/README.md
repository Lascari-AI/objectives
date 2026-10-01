# Artifacts

Every output this objective produces lands here. This example ships no real results.

- `baseline/`: `baseline_summary.json` and `parsed_output.jsonl` from phase 1.
- `profiles/`: one profile file per profiled run.
- `candidates.md`: one row per candidate with its mechanism, decision, and reason.
- `runs/<run_id>/`: one folder per benchmark run.
    - `run_manifest.json`, `timings.csv`, `summary.json`, `status.json`, `run.log.jsonl`.
    - Rejected runs stay here. They are evidence for the decision.
- `final_comparison.md`: baseline versus the combined change, written in phase 4.
