# Documentation Review

Initial documentation draft: September 18, 2026, branch `docs/objectives-first-draft`. The native corpus is `docs/`, registered by `codecaine.docs.json`. It has five pages: Foundation, System Design, Agents, Implementation, and a create/resume guide.

Open the local editable overview:

http://127.0.0.1:4820/projects/objectives-e72003eb/docs/#/00-foundation

The workbench edits the native document body directly and autosaves. Use the connected typed Codecaine Docs tools for agent edits; do not hand-edit `doc.json` or component sidecars. Generated indexes and local change journals are ignored.

## Open the Workbench

This checkout currently uses the local Codecaine Docs source installation and Bun:

```sh
bun /Users/Ford/workspace/codecaine/core/docs-system/packages/docs-mcp/src/cli.ts ui \
  --workspace /Users/Ford/workspace/codecaine/agent-config/objectives \
  --project objectives-e72003eb
```

This discovers the corpus and opens it through the shared Docs service. Rediscover the project ID if the checkout moves. These absolute paths describe the current local setup, not a portable package installation.

## Check a Static Export

```sh
bun /Users/Ford/workspace/codecaine/core/docs-system/packages/docs-cli/src/index.ts export \
  --root /Users/Ford/workspace/codecaine/agent-config/objectives/docs \
  --out /tmp/objectives-static-review
```

The first draft passed native validation, link checking, and static export. GitHub Pages is the intended public destination; no Pages workflow or deployment was added in this draft pass.

## Companion Article

Site preview: http://127.0.0.1:4196/blog/drafts/objectives/

Blog editor: http://127.0.0.1:4820/projects/personal-site-writing-38daf15d/docs/#/objectives

The article is in the personal-site repository on `writing/decomp-autohdr-objectives`. It remains unpublished, with local docs links for review. Replace those links with verified public destinations before publication.
