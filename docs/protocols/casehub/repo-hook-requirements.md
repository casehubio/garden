---
id: PP-20260525-c8c05d
title: "All casehub repos with Work Tracking must have both pre-push and commit-msg hooks in .githooks/"
type: rule
scope: platform
applies_to: "all casehub project repos where CLAUDE.md declares 'Issue tracking: enabled'"
severity: important
refs:
  - docs/protocols/universal/committed-git-hooks.md
violation_hint: ".githooks/pre-push or .githooks/commit-msg missing in a repo with Work Tracking enabled"
created: 2026-05-25
---

Any casehub repo with `Issue tracking: enabled` in its `## Work Tracking` CLAUDE.md section must have both hooks committed to `.githooks/`: `pre-push` (from git-squash skill — detects squash candidates before push) and `commit-msg` (from issue-workflow skill — blocks commits without `Refs #N`, `Closes #N`, or `no-issue` bypass). Install via `workspace-init` Step 7c, which handles both hooks in a single commit. On fresh clones, activate with `git config core.hooksPath .githooks`.
