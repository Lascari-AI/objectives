# Examples

Two finished objective bundles, each paused mid-run so you can see a real handoff.

- [`parser-benchmark/`](parser-benchmark/): a simple single-metric objective.
    - Shows the guide's example goal filled out end to end: fixed fixture, edit scope, a five-phase plan with gates, and a state file with one rejected experiment and a named next action.
- [`image-grouping-accuracy/`](image-grouping-accuracy/): a multi-dataset research objective from a public image-grouping challenge.
    - Shows why the goal names every metric on every dataset: "get the accuracy higher" lets an agent stop after one dataset improves.
    - Also shows image-content-only rules, dev and holdout shards, MLX for research scripts only, sub-agent visual review, and an active run handoff.
- Commands, paths, and measurements are illustrative. Replace them with verified values from your own project.
- Paths inside the bundles use `objectives/<slug>/`, as they would in a target repo.
