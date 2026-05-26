---
id: PP-20260526-849f45
title: "Each casehub project workspace must be an isolated git repo"
type: rule
scope: platform
applies_to: "All casehub project workspaces under $CASEHUB_WORKSPACE"
severity: important
refs:
  - docs/new-repo-checklist.md#14-workspace-setup
violation_hint: "git rev-parse --show-toplevel from a workspace subdir returns the parent workspace path instead of the subdir path — workspace sessions bleed branches and commits into the parent repo"
created: 2026-05-26
---

Every project workspace directory (e.g. `$CASEHUB_WORKSPACE/<name>/`) must be initialised with `git init` and backed by its own GitHub repo (`wsp-casehub-<name>`, private, under the developer's personal account). Never commit a workspace subdirectory into the parent workspace git repo. After initialisation, add `/<name>` to the parent workspace `.gitignore` so the child repo does not appear as untracked noise in parent `git status`.
