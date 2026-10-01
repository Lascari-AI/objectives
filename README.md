# Objectives

A lightweight way to keep a coding agent on track for long-running tasks. An objective is a prompt and a file structure: a short goal with explicit stopping criteria, context files the agent rereads after every compaction, a workspace for every script and run output, and a current-state file it resumes from. No service, no scheduler, nothing to deploy.

- **Docs:** https://lascari-ai.github.io/objectives/
- **Skill:** [`.codex/skills/setup-objective/SKILL.md`](.codex/skills/setup-objective/SKILL.md)
- **Examples:** [`examples/`](examples/) has finished objective bundles you can copy.

## Quick Start

```bash
git clone https://github.com/Lascari-AI/objectives.git ~/tools/objectives
cd /path/to/your/project
python3 ~/tools/objectives/.codex/skills/setup-objective/scripts/create_objective.py my-objective
```

Then ask your agent to use the setup-objective skill to fill in the objective, and start a goal that points at it. The [create and resume guide](https://lascari-ai.github.io/objectives/#/40-guides) walks through the rest.

## License

MIT. See [LICENSE](LICENSE).
