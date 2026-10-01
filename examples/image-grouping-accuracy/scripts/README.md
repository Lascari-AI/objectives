# Scripts

Objective-local research code goes here. This example ships no real code.

- `build_feature_cache.py`: computes image-content pair features once per shard and saves them to `artifacts/feature_cache/`.
- `proxy_sweep_mlx.py`: scores many configs against cached features.
    - Uses MLX on the local Apple GPU by default. `--backend numpy` runs the same math on CPU.
    - Supports `--status`, `--resume`, and `--dry-run-resume-plan`, and reads `control.json` while running.
- `gate_table.py`: turns per-shard scores into mean, p10, median, and worst per dataset, and marks each of the 12 gates.
- `render_failures.py`: renders false merge, false split, and control sheets for sub-agent visual review.
- `audit_leakage.py`: scans `pipeline/grouping/` for reads of filenames, file times, capture order, shard ids, or truth labels.
- `requirements-research.txt`: research-only dependencies such as `mlx`.
    - MLX stays here. It never goes into the production requirements.
