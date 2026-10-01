<goal>
- Reduce parser benchmark median runtime by at least 15 percent without changing parsed output or increasing peak memory.
- This objective owns the parser hot path in `src/parser/` and its own workspace under `objectives/parser-benchmark/`.
</goal>

<context_refresh>
- Reread objectives/parser-benchmark/goal.md.
- Reread objectives/parser-benchmark/current_state.md.
- Reread the relevant objectives/parser-benchmark/context/*.md files.
- Reread these files at the start of work and after every compaction or resume.
</context_refresh>

<working_strategy>
- Reproduce the fixed baseline, profile the bottleneck, change one mechanism, and compare repeated runs.
- Run each experiment against the same fixture, interpreter, and repeat count as the baseline.
- Keep benchmark scripts in `objectives/parser-benchmark/scripts/` and every run output in `objectives/parser-benchmark/artifacts/`.
- Follow the phases and gates in `context/03_working_plan.md`.
</working_strategy>

<success_metrics>
- Lower median runtime with output equivalence and peak memory within the baseline.
- Every experiment has a run folder with its command, code revision, measurements, and an accept or reject decision.
</success_metrics>

<non_goals>
- Do not change the fixture, public API, or evaluator to improve the result.
- Do not add new runtime dependencies or native extensions.
- Do not tune for one machine with settings that would not ship.
</non_goals>

<completion_criteria>
- Median runtime on the fixed fixture is at least 15 percent below the recorded baseline.
- Parsed output is identical to the baseline output.
- Peak memory is at or below the measured baseline.
- Commands, code and fixture identities, measurements, and the final decision are recorded.
- current_state.md records accepted work and any remaining limits.
</completion_criteria>
