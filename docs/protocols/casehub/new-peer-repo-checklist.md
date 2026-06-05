---
id: PP-20260605-54f327
title: "New casehub peer repo must satisfy the full creation checklist before first commit"
type: rule
scope: platform
applies_to: "Any session creating a new peer repo in the casehub ecosystem"
severity: important
refs:
  - casehub/repo-hook-requirements.md
violation_hint: "Repo created on GitHub but missing one or more of: CLAUDE.md with Work Tracking, .githooks/pre-push, .github/workflows/publish.yml, entry in parent CLAUDE.md peer list, PLATFORM.md entries, workspace repo"
created: 2026-06-05
---

When a new peer repo is added to the casehub ecosystem, all of the following must be in place before the first substantive commit: (1) `CLAUDE.md` with project type, module structure, key rules, Work Tracking enabled, and GitHub repo set; (2) `.githooks/pre-push` copied from an existing peer (e.g. `casehub-connectors`); (3) `.github/workflows/publish.yml` with upstream-published trigger, build/test/publish steps, and downstream dispatch to consuming repos; (4) entry in `casehubio/parent` CLAUDE.md peer repos list; (5) `PLATFORM.md` entries: module registry row, build/dependency order, capability ownership row(s), and per-repo deep-dive pointer; (6) `mdproctor/wsp-casehub-<name>` workspace repo (private → public) with standard directories and HANDOFF.md pointing to the first session's work. Missing any item leaves the repo invisible to CI/CD, session discovery, or the build graph.
