<constraints>
    <hard_rules>
        - Production rules infer groups from image content only.
            - Allowed: pixels and features computed from pixels, such as image size, color histograms, structural similarity, edges, and embeddings.
        - One production config must pass all 12 gates. Per-dataset configs do not count.
        - Every gate is measured on the full shard set of its dataset, including holdout shards.
        - Production code under `pipeline/` stays CPU-only and must run on a machine without a GPU.
    </hard_rules>

    <forbidden_shortcuts>
        - Using filenames, file paths, file modification times, EXIF capture order, or shard ids in a rule is invalid because real input does not carry reliable order or names.
        - Using truth labels or reviewed visual labels as rule inputs is invalid because they do not exist at inference time.
        - Tuning on holdout shards is invalid because it hides overfitting.
        - Hard-coding image ids or cases found in visual review is invalid because it fits the sample, not the pattern.
        - Editing shards, truth labels, or the scorer is invalid because it changes what is measured.
    </forbidden_shortcuts>

    <data_and_feature_boundaries>
        - Deployable: image pixels and features computed from them.
        - Diagnostic only: truth labels, per-shard scores, false merge and false split lists, and visual review notes. Use them to choose experiments, never as rule inputs.
        - Shard split, fixed for the whole objective:
            - Dev shards: listed in `data/<dataset>/splits/dev.txt`. Use for sweeps and tuning.
            - Holdout shards: listed in `data/<dataset>/splits/holdout.txt`. Score only for finalists, at most once per finalist.
        - Research only: MLX on the local Apple GPU, for scripts under `objectives/image-grouping-accuracy/scripts/`.
    </data_and_feature_boundaries>

    <risk_budget>
        - Worst shard: a finalist may not lower the worst shard of any dataset below its baseline.
        - False merges: a candidate that adds more than 10 percent false merges on any dataset is rejected, even if its mean rises.
        - Holdout gap: if a finalist's holdout mean is more than 0.15 points below its dev mean on any dataset, treat it as overfit and reject it.
    </risk_budget>

    <promotion_or_completion_gates>
        - `gate_table`: all 12 gates in `goal.md` pass for one config.
        - `leakage_audit`: `audit_leakage.py` finds no forbidden field read anywhere in `pipeline/grouping/`.
        - `cpu_parity`: the promoted config gives the same groups with and without MLX installed.
        - `tests`: `pytest tests/grouping` passes on CPU.
    </promotion_or_completion_gates>
</constraints>
