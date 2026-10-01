# Artifacts

Every output this objective produces lands here. This example ships no real results.

- `baseline/`: per-shard scores and the baseline gate table for all four datasets.
- `feature_cache/<dataset>/`: cached pair features for dev shards, with fingerprints.
- `config_matrix.csv`: every candidate config, its family, hypothesis, and decision.
- `runs/<run_id>/`: one folder per sweep or validation run.
    - `run_manifest.json`, `status.json`, `progress.jsonl`, `run.log.jsonl`, `checkpoint.json`, `control.json`, `outputs/`.
- `visual_review/<round_id>/`: rendered sheets, raw sub-agent reports, and `synthesis.csv`.
- `gate_table.csv`: the 12 gates for each finalist, split by dev and holdout.
- `promotion/`: leakage audit, CPU test log, and CPU parity table for the promoted config.
