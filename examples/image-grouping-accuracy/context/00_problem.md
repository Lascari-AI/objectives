<problem>
    <objective_question>
        - Can one production config of the grouping pipeline clear all 12 gates across random10, random20, random40, and random80 using image content only?
    </objective_question>

    <current_baseline>
        - The task: given a folder of property photos, put the images of each scene into one group.
            - A scene is often several bracketed exposures of the same view, so groups mix dark, normal, and bright frames.
        - The score for one shard is exact-group accuracy.
            - A truth group counts only if one predicted group contains exactly its images.
            - Score is exact truth groups divided by all truth groups.
        - Each dataset is split into shards, and each gate is computed over that dataset's per-shard scores.
            - `mean`: average shard score.
            - `p10`: 10th percentile shard score.
            - `median`: middle shard score.
            - `worst`: lowest shard score.
        - Baseline scores live in `artifacts/baseline/gate_table.csv` once phase 1 runs.
    </current_baseline>

    <why_current_state_is_insufficient>
        - A single target such as "get the accuracy higher" lets an agent stop after one dataset improves.
        - Datasets fail in different ways.
            - Small shards are hurt most by a few false merges.
            - Large shards have more lookalike rooms, so false merges and long chains grow.
        - Mean alone hides bad shards. The p10, median, and worst gates force the tail to improve too.
    </why_current_state_is_insufficient>

    <failure_modes>
        - `false_merge`: two scenes join into one group. Common with same-room lookalikes and repeated layouts.
        - `false_split`: one scene breaks into several groups. Common with exposure extremes, clipping, and small viewpoint shifts.
        - `chain_merge`: a run of weak links joins many scenes into one large group.
        - `shard_overfit`: a config tuned on a few shards wins there and loses on unseen shards.
        - `metadata_leak`: a rule uses filenames, file times, or capture order. It scores well on a local dataset and fails on real input.
    </failure_modes>

    <prior_evidence>
        - `artifacts/baseline/per_shard_scores.csv`: baseline score per shard, with exact, false merge, and false split counts.
        - `artifacts/visual_review/`: sub-agent reports and the structured synthesis from each review round.
    </prior_evidence>

    <expected_value>
        - Passing all 12 gates means the pipeline holds up across shard sizes, not only on the easiest one.
        - A config that passes some gates is useful evidence, but it does not complete the objective.
    </expected_value>
</problem>
