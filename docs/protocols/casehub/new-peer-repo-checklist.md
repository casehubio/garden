---
id: PP-20260605-54f327
title: "New casehub peer repo must satisfy the full creation checklist before first commit"
type: rule
scope: platform
applies_to: "Any session creating a new peer repo in the casehub ecosystem"
severity: important
refs:
  - casehub/repo-hook-requirements.md
violation_hint: "Repo created on GitHub but missing one or more of: CLAUDE.md with routing/artifact tables, .githooks/pre-push, hooksPath configured, .github/workflows/publish.yml, entry in parent CLAUDE.md peer list, PLATFORM.md entries, Maven module skeletons, proj/ symlink, workspace repo with HANDOFF.md"
created: 2026-06-05
---

When a new peer repo is added to the casehub ecosystem, all of the following must be in place before the first substantive commit:

(1) `CLAUDE.md` with project type, module structure, key rules, `## Artifact Locations` table, `## Routing` table, Work Tracking enabled, and GitHub repo set.

(2) `.githooks/pre-push` copied from an existing peer (e.g. `casehub-connectors`) and activated: `git config core.hooksPath .githooks`.

(3) `.github/workflows/publish.yml` with upstream-published trigger, build/test/publish steps, and downstream dispatch to consuming repos.

(4) Entry in `casehubio/parent` CLAUDE.md peer repos list.

(5) `PLATFORM.md` entries: module registry row, build/dependency order, capability ownership row(s), and per-repo deep-dive pointer.

(6) **Maven module skeleton** — for each declared module in `pom.xml`: create the directory, `src/main/java/`, `src/test/java/`, and a minimal `pom.xml` inheriting from the parent. Without these, `mvn install` fails before a single line of code is written.

(7) `mdproctor/wsp-casehub-<name>` workspace repo (public) with standard directories (`blog/`, `plans/`, `specs/`, `design/`, `snapshots/`, `adr/`), `IDEAS.md`, and `HANDOFF.md` pointing to the first session's work.

(8) Bidirectional symlinks between project and workspace — none committed to git, all in `.gitignore`:

   **In the project repo** (add `wksp` to project `.gitignore`):
   - `wksp` → workspace: `ln -s ~/claude/public/casehub-<name> ~/claude/casehub/<name>/wksp`

   **In the workspace** (add `proj` and `CLAUDE.md` to workspace `.gitignore`):
   - `proj` → project repo: `ln -s ~/claude/casehub/<name> ~/claude/public/casehub-<name>/proj`
   - `CLAUDE.md` → project CLAUDE.md: `ln -s ~/claude/casehub/<name>/CLAUDE.md ~/claude/public/casehub-<name>/CLAUDE.md`

   Without `proj`, `work-start` cannot resolve workspace vs project paths. Without `CLAUDE.md`, Claude Code running from the workspace cannot find project conventions. Without `wksp`, skills running from the project repo cannot find the workspace.

Missing any item leaves the repo invisible to CI/CD, session discovery, the build graph, or unable to start a proper work session.
