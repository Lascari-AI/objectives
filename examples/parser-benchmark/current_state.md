<current_state>
<last_updated>2026-05-12</last_updated>

<status>
- In progress. Phase 3 of 5, single mechanism experiments.
- Baseline is recorded. One experiment is rejected. No change is accepted yet.
- All numbers in this file are illustrative.
</status>

<completed>
- Phase 1, baseline reproduction.
    - `baseline-a` and `baseline-b` agree within 1.4 percent.
    - Baseline median 2.84 s, IQR 0.05 s, peak RSS 212 MB.
    - Canonical parsed output saved for both fixtures.
- Phase 2, bottleneck profile.
    - `Lexer.next_token` takes 46 percent of pass time, mostly string slicing.
    - `coerce_scalar` takes 19 percent, one regex match per scalar.
    - Three candidates ranked in `artifacts/candidates.md`.
- Rejected `c1-scalar-cache`.
    - Cached `coerce_scalar` results by raw string.
    - Median dropped 6.1 percent, but peak RSS rose to 231 MB, which fails the memory gate.
    - Output was identical and tests passed.
</completed>

<in_progress>
- Nothing running. Next candidate is `c2-lexer-index-scan`.
    - Mechanism: scan the input with an index into one string instead of slicing a new string per character run.
    - Target: `Lexer.next_token` in `src/parser/lexer.py`.
</in_progress>

<next_actions>
- Implement `c2-lexer-index-scan` on branch `parser-bench/c2-lexer-index-scan`.
- Then run `pytest tests/parser`, the benchmark with run id `c2-lexer-index-scan-r1`, and `compare_output.py`.
- Record the decision in `artifacts/candidates.md` and here.
</next_actions>

<risks_or_open_questions>
- The 15 percent target may need two accepted changes. If `c2` alone falls short, phase 4 has to combine it with `c3-fast-int-path`.
- `c1` might pass with a bounded cache. Retry it only after `c2` and `c3` are decided, and only if the peak still stays at or below 212 MB.
</risks_or_open_questions>

<important_paths>
- `objectives/parser-benchmark/goal.md`
- `objectives/parser-benchmark/context/03_working_plan.md`
- `objectives/parser-benchmark/scripts/`: `bench_parser.py`, `compare_output.py`, `profile_parser.py`.
- `objectives/parser-benchmark/artifacts/baseline/baseline_summary.json`
- `objectives/parser-benchmark/artifacts/baseline/parsed_output.jsonl`
- `objectives/parser-benchmark/artifacts/profiles/baseline_profile.txt`
- `objectives/parser-benchmark/artifacts/candidates.md`
- `objectives/parser-benchmark/artifacts/runs/c1-scalar-cache-r1/summary.json`: rejected run.
</important_paths>

<active_runs>
- None. No benchmark is running.
</active_runs>
</current_state>
