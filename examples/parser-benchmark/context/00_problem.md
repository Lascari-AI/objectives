<problem>
    <objective_question>
        - Can the config parser run at least 15 percent faster on the fixed benchmark fixture with identical output and no extra peak memory?
    </objective_question>

    <current_baseline>
        - The baseline is the parser at the commit recorded in `artifacts/baseline/baseline_summary.json`.
        - The benchmark parses every file in `fixtures/bench/configs_large/` and reports wall time per full pass.
            - 1,200 config files, about 48 MB in total.
            - The fixture fingerprint is a SHA-256 over sorted file paths and contents, stored in the baseline summary.
        - Baseline metrics are median runtime over 15 timed passes after 3 warmup passes, plus peak resident memory for one pass.
    </current_baseline>

    <why_current_state_is_insufficient>
        - Config loading runs at every service start and in every CI job, so parser time adds directly to startup and test time.
        - An earlier profile showed most time in the lexer, not in file I/O.
            - Token scanning builds many short-lived strings.
            - Value conversion runs a regex match per scalar.
    </why_current_state_is_insufficient>

    <failure_modes>
        - `noise_win`: A change looks faster because of machine noise. Repeated runs and a spread check guard against this.
        - `output_drift`: A faster path changes parsed values, key order, or error messages. Output equivalence catches this.
        - `memory_trade`: A cache speeds up parsing by holding more memory. The peak memory gate rejects it.
        - `fixture_overfit`: A change is tuned to the fixture's file shapes and slows real configs. The tests and a second sanity fixture guard against this.
    </failure_modes>

    <prior_evidence>
        - `artifacts/profiles/baseline_profile.txt`: cProfile output for one pass, sorted by cumulative time.
        - `artifacts/baseline/baseline_summary.json`: baseline median, spread, peak memory, revision, and fixture fingerprint.
    </prior_evidence>

    <expected_value>
        - A 15 percent drop in median runtime is large enough to show up in service start time and CI wall time.
        - A smaller verified gain is still worth recording, but it does not complete the objective.
    </expected_value>
</problem>
