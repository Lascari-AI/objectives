<goal>
- Raise exact-group accuracy of the image grouping pipeline until all four datasets (random10, random20, random40, random80) clear every gate in completion_criteria.
- This objective owns `pipeline/grouping/` and its workspace under `objectives/image-grouping-accuracy/`.
- One dataset improving is progress, not completion.
</goal>

<context_refresh>
- Reread objectives/image-grouping-accuracy/goal.md.
- Reread objectives/image-grouping-accuracy/current_state.md.
- Reread the relevant objectives/image-grouping-accuracy/context/*.md files.
- Reread these files at the start of work and after every compaction or resume.
</context_refresh>

<working_strategy>
- Reproduce the baseline on every shard of all four datasets before changing behavior.
- Tune on dev shards only. Score holdout shards only for finalists.
- Screen candidates with cheap proxy sweeps, then fully validate a small finalist set on all four datasets.
- Research scripts may use MLX on the local GPU. Production code stays CPU-only and portable.
- Have a sub-agent review rendered failure sheets and return observation-only reports. Turn each recurring pattern into a testable feature hypothesis built from image content.
- Follow the phases and gates in `context/03_working_plan.md`.
</working_strategy>

<success_metrics>
- Per-dataset gate table in `artifacts/gate_table.csv` moves toward all 12 gates passing.
- No finalist regresses the worst shard of any dataset below baseline.
- Every candidate links to its config row, per-shard results, and accept or reject reason.
</success_metrics>

<non_goals>
- Do not use filenames, file times, EXIF capture order, shard ids, or truth labels in production rules.
- Do not edit shards, truth labels, or the scorer.
- Do not add MLX or any GPU dependency to `pipeline/`.
- Do not stop because one dataset or one metric improved.
</non_goals>

<completion_criteria>
- All gates pass on the full shard set of each dataset, using one production config:
    - random10: mean >= 99.40%, p10 >= 99.25%, worst >= 99.05%
    - random20: mean >= 99.35%, p10 >= 99.15%, worst >= 99.00%
    - random40: mean >= 99.25%, median >= 99.20%, worst >= 98.95%
    - random80: mean >= 99.15%, median >= 99.10%, worst >= 98.90%
- The production pipeline infers groups from image content only. The leakage audit passes.
- Production tests pass on CPU with no MLX installed.
- Config, code revision, per-shard scores, and the gate table are recorded in `artifacts/`.
- current_state.md records the final gate table, accepted changes, and remaining limits.
- If any gate fails, the objective is not complete.
</completion_criteria>
