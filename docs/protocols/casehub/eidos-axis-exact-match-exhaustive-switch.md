---
id: PP-20260607-7dd3ce
title: "axisExactMatch must use exhaustive switch on DispositionAxis with no default branch"
type: rule
scope: repo
applies_to: "casehub-eidos: any VocabularyTerm implementation overriding axisExactMatch(Class<?>, DispositionAxis)"
severity: important
refs:
  - casehubio/eidos#40
violation_hint: "return Optional.of(switch (axis) { ... }) with no gap branches; switch with a default-> arm; axisExactMatch that iterates a map instead of switching"
created: 2026-06-07
---

When a `VocabularyTerm` implementation covers a target vocabulary in `axisExactMatch()`, the switch on `DispositionAxis` must be exhaustive with no default branch. `Optional.empty()` is a valid branch for axes where no meaningful mapping exists. Do not wrap the switch in `Optional.of(switch {...})` — that forbids gaps and forces invented mappings. The no-default exhaustive switch is the compile-time completeness gate: adding `CONFLICT_MODE` (or any future value) to `DispositionAxis` causes a compile error at every `axisExactMatch` implementation that covers a target vocabulary, forcing explicit handling of the new axis.
