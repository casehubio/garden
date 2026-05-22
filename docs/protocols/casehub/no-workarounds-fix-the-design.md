---
id: PP-20260522-3b1ccd
title: "Fix the design — no workarounds or backward-compatibility shims"
type: principle
scope: platform
applies_to: "All design and implementation work across CaseHub modules"
severity: important
refs:
  - casehub/parent/docs/prompt-snippets.md
violation_hint: "Adding a wrapper class, compatibility adapter, or shim to preserve an existing API instead of breaking callers and fixing them"
created: 2026-05-22
---

This platform has no end users. Breaking method signatures, renaming modules, or
restructuring SPIs costs nothing externally — the migration is mechanical and every
caller is made explicit. When the right design requires a change that breaks existing
call sites, make the change. Workarounds, wrappers, and backward-compatibility shims
are not acceptable alternatives — they add layers with no architectural benefit and
require unwinding later at greater cost. The only exception is when the session
explicitly grants permission to preserve a specific interface for this task.
