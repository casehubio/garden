---
id: PP-20260524-d99f74
title: "Use org/repo format in CI REPOS lists when the ecosystem spans multiple GitHub orgs"
type: rule
scope: universal
applies_to: "Any CI workflow that iterates over a repo list spanning more than one GitHub org"
severity: important
refs: []
violation_hint: "Bare repo name in REPOS list assumes a single org and silently excludes or breaks repos in other orgs; cloning a cross-org repo with GITHUB_TOKEN returns 404"
created: 2026-05-24
---

When a repository ecosystem spans more than one GitHub org (e.g. a personal fork, a partner org, or an app repo hosted separately), CI workflow REPOS lists must use `org/repo` format (`acme/ledger`, `personal/quarkmind`) rather than bare repo names. All API calls and link construction then use the full ref directly, eliminating the implicit single-org assumption. Cloning repos outside the workflow's home org additionally requires a classic PAT with cross-org read access (`GH_PAT`) — `GITHUB_TOKEN` is scoped to the triggering repo's org only and returns 404 on cross-org clone attempts.
