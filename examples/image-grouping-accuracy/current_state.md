<current_state>
<last_updated>2026-05-12</last_updated>

<status>
- In progress. Second loop of phases 3 to 5.
- Best finalist `cfg-0412` passes all three random80 gates on dev. random10, random20, and random40 still fail.
- A proxy sweep for round 3 is running on the local GPU.
- All measurements in this file are illustrative. Only the gates are real.
</status>

<completed>
- Phase 1, baseline on every shard of all four datasets. Gate table in `artifacts/baseline/gate_table.csv`.
- MLX and NumPy parity check passed on two random10 dev shards.
- Round 1 sweep and full validation.
    - `cfg-0412` adds an exposure-bracket rescue that joins dark and bright frames with matching edge layout.
    - Illustrative dev gate table for `cfg-0412`:
        - random10: mean 99.31% (fail), p10 99.18% (fail), worst 99.07% (pass)
        - random20: mean 99.29% (fail), p10 99.16% (pass), worst 98.94% (fail)
        - random40: mean 99.22% (fail), median 99.21% (pass), worst 98.97% (pass)
        - random80: mean 99.17% (pass), median 99.12% (pass), worst 98.92% (pass)
- Round 2 visual review on `cfg-0412` failures.
    - Sub-agent reports found two recurring patterns in new false merges.
        - Same-room lookalikes: different corners of one room with matching color histograms.
        - Repeated layouts: matching bathrooms or bedrooms in one listing.
    - Synthesis maps both to a brake hypothesis: require structural similarity and a dimension match before a histogram-led merge.
- Rejected `cfg-0388`. It raised random10 mean but added 14 percent more false merges on random40, over the 10 percent budget.
</completed>

<in_progress>
- Round 3 proxy sweep `sweep-r3` on dev shards, 4,096 configs.
    - Families: the new structural brake, with and without the exposure rescue, plus ablations.
</in_progress>

<next_actions>
- Check `sweep-r3` with `proxy_sweep_mlx.py --status`.
- When it finishes, read `finalists.csv` and run full validation on dev for all four datasets.
- Rank finalists by weakest-gate margin across all 12 gates. Reject any that loses a random80 gate.
</next_actions>

<risks_or_open_questions>
- random10 needs fewer false merges and random20 worst needs fewer false splits. One brake may help the first and hurt the second.
- random20 worst is set by a single shard. Check whether it has a data issue before tuning hard against it.
- Holdout has not been scored for any finalist yet. No gate counts as passed until holdout passes too.
</risks_or_open_questions>

<important_paths>
- `objectives/image-grouping-accuracy/goal.md`
- `objectives/image-grouping-accuracy/context/03_working_plan.md`
- `objectives/image-grouping-accuracy/scripts/`: `build_feature_cache.py`, `proxy_sweep_mlx.py`, `gate_table.py`, `render_failures.py`, `audit_leakage.py`.
- `objectives/image-grouping-accuracy/artifacts/baseline/gate_table.csv`
- `objectives/image-grouping-accuracy/artifacts/gate_table.csv`: latest finalist gates.
- `objectives/image-grouping-accuracy/artifacts/config_matrix.csv`
- `objectives/image-grouping-accuracy/artifacts/visual_review/round-2/synthesis.csv`
- `objectives/image-grouping-accuracy/artifacts/runs/sweep-r3/`
</important_paths>

<active_runs>
    <run id="sweep-r3">
        <command>
            - `python objectives/image-grouping-accuracy/scripts/proxy_sweep_mlx.py --matrix objectives/image-grouping-accuracy/artifacts/config_matrix.csv --run-dir objectives/image-grouping-accuracy/artifacts/runs/sweep-r3`
        </command>
        <pid_or_session>
            - `PID 48213` (illustrative)
        </pid_or_session>
        <started_at>
            - `2026-05-12T14:05:00-05:00`
        </started_at>
        <status_path>
            - `objectives/image-grouping-accuracy/artifacts/runs/sweep-r3/status.json`
        </status_path>
        <log_path>
            - `objectives/image-grouping-accuracy/artifacts/runs/sweep-r3/run.log.jsonl`
        </log_path>
        <checkpoint_path>
            - `objectives/image-grouping-accuracy/artifacts/runs/sweep-r3/checkpoint.json`
        </checkpoint_path>
        <control_path>
            - `objectives/image-grouping-accuracy/artifacts/runs/sweep-r3/control.json`
        </control_path>
        <safe_next_action>
            - Read `status.json`. If the heartbeat is under 5 minutes old, wait.
            - If it is older, check the last 100 log events, then run `proxy_sweep_mlx.py --dry-run-resume-plan --run-dir objectives/image-grouping-accuracy/artifacts/runs/sweep-r3` before resuming.
        </safe_next_action>
    </run>
</active_runs>
</current_state>
