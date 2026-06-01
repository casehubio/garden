---
id: PP-20260601-057cde
title: "CI dispatch chain must cover all direct Maven consumers"
type: rule
scope: platform
applies_to: "all casehubio repos — every 'Trigger downstream CI' step in any GitHub Actions workflow"
severity: important
violation_hint: "A repo declares casehub-X as a compile/runtime dependency in pom.xml but is absent from casehub-X's downstream dispatch list"
created: 2026-06-01
---

When a casehubio repo publishes to GitHub Packages, its workflow must dispatch `upstream-published` to every repo that declares it as a direct Maven compile/runtime dependency. Transitive consumers are covered through chaining — only direct pom.xml dependencies need to be in the dispatch list. Omitting a direct consumer causes silent CI failure: that repo never rebuilds when its dependency changes. Audit: for each candidate repo, grep its pom.xml for the publishing repo's artifact ID and verify the repo appears in the dispatch list.
