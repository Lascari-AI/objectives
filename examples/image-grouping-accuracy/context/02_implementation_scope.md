<implementation_scope>
    <owned_surfaces>
        - `pipeline/grouping/features.py`: image-content features. Add or change feature columns here.
        - `pipeline/grouping/rules.py`: merge, split, and brake rules that turn pair features into groups.
        - `pipeline/grouping/config/default.json`: production thresholds. Change only through promotion in phase 6.
        - `objectives/image-grouping-accuracy/scripts/`: research scripts, sweeps, renderers, and audits.
        - `objectives/image-grouping-accuracy/artifacts/`: every output this objective produces.
    </owned_surfaces>

    <read_only_references>
        - `data/<dataset>/shards/<shard_id>/images/`: shard images. Never edit.
        - `data/<dataset>/shards/<shard_id>/truth.json`: truth groups. Read only by the scorer and diagnostic scripts.
        - `data/<dataset>/splits/`: dev and holdout shard lists. Fixed for the objective.
        - `eval/score.py`: exact-group scorer. Never edit.
    </read_only_references>

    <generated_outputs>
        - `artifacts/baseline/`: per-shard scores and the baseline gate table.
        - `artifacts/feature_cache/<dataset>/<shard_id>.npz`: cached pair features for proxy sweeps.
        - `artifacts/config_matrix.csv`: every candidate config.
        - `artifacts/runs/<run_id>/`: sweep and validation runs with status, log, checkpoint, and control files.
        - `artifacts/visual_review/<round_id>/`: rendered sheets, sub-agent reports, and `synthesis.csv`.
        - `artifacts/gate_table.csv`: latest gate table for each finalist.
    </generated_outputs>

    <commands_and_entrypoints>
        - `python eval/run_shards.py --dataset <name> --split dev --config <config.json> --out <run_dir>`: runs the production pipeline on a shard split and scores it.
        - `python objectives/image-grouping-accuracy/scripts/build_feature_cache.py --dataset <name> --split dev`: caches pair features once per shard.
        - `python objectives/image-grouping-accuracy/scripts/proxy_sweep_mlx.py --matrix artifacts/config_matrix.csv --run-dir <run_dir>`: scores many configs against cached features on the local GPU.
            - `--status` prints progress. `--resume` continues from the checkpoint. `--backend numpy` runs the same sweep on CPU.
        - `python objectives/image-grouping-accuracy/scripts/gate_table.py --run-dir <run_dir>`: computes mean, p10, median, and worst per dataset and marks each gate.
        - `python objectives/image-grouping-accuracy/scripts/render_failures.py --run-dir <run_dir> --out artifacts/visual_review/<round_id>`: renders false merge and false split sheets.
        - `python objectives/image-grouping-accuracy/scripts/audit_leakage.py pipeline/grouping`: checks production code for forbidden fields.
    </commands_and_entrypoints>

    <adjacent_surfaces_requiring_caution>
        - `pipeline/io/`: image loading. Changes here touch every stage. Edit only to fix a bug, and record it in state.
        - `requirements.txt`: production dependencies. MLX must not appear here. Research dependencies go in `objectives/image-grouping-accuracy/scripts/requirements-research.txt`.
    </adjacent_surfaces_requiring_caution>

    <out_of_scope>
        - Relabeling truth or rebuilding shards. That is a data objective, not a grouping objective.
        - Moving production inference to the GPU.
        - Changing the score definition.
    </out_of_scope>
</implementation_scope>
