---
id: PP-20260525-299999
title: "Commit git hooks to .githooks/ with core.hooksPath — never install to .git/hooks/"
type: rule
scope: universal
applies_to: "all git repositories receiving shared hooks"
severity: important
refs:
  - docs/protocols/casehub/repo-hook-requirements.md
violation_hint: "Hook file exists at .git/hooks/pre-push or .git/hooks/commit-msg instead of .githooks/"
created: 2026-05-25
---

Git hooks installed to `.git/hooks/` are machine-local and disappear on clone. Commit hooks to `.githooks/` in the repo root (tracked by git), then activate with `git config core.hooksPath .githooks`. This makes hooks visible to all contributors automatically on clone — the only per-machine step is running `git config core.hooksPath .githooks` once. Hook files belong in `.githooks/`; `core.hooksPath` config belongs in local `.git/config` (not committed).
