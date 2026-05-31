---
id: PP-20260531-a18973
title: "Sweep items with blockers must be flagged, documented, and removed — not silently deferred"
type: rule
scope: platform
applies_to: "sweep branches — S/XS umbrella issues spanning multiple small items"
severity: important
violation_hint: "item skipped without a comment, left partially attempted, or removed from scope with no label or blocker documentation"
created: 2026-05-31
---

When a sweep branch encounters an item with unresolved design questions, missing dependencies, or external blockers, four steps are mandatory before moving on: add a `blocked` label on GitHub; escalate complexity to `High` if currently rated lower; leave a comment on the issue enumerating each specific blocker with enough detail for a future session to act without re-investigation; and note the removal in the sweep umbrella issue with a reference back to the blocking comment. The item is never silently skipped, left partially attempted, or removed from scope without documentation. "S · Low" labels describe anticipated difficulty — they do not override real blockers discovered during execution.
