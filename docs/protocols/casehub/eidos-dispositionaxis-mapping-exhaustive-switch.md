---
id: PP-20260609-9eadab
title: "All DispositionAxis mapping helpers must use exhaustive switch with no default branch"
type: rule
scope: repo
applies_to: "casehub-eidos: any method that maps DispositionAxis to a concrete value — JSON key, display label, payload field, or structural rendering token"
severity: important
refs:
  - docs/superpowers/specs/2026-06-08-framework-grounding-design.md
violation_hint: "switch (axis) { ... default -> ... } in any helper that maps DispositionAxis to String or similar; using a Map<DispositionAxis, String> lookup instead of a switch"
created: 2026-06-09
---

Any method that maps a `DispositionAxis` value to a concrete output — a JSON key (`axisJsonKey`), a display label (`axisLabel`), a structural rendering token — must use an exhaustive switch expression with no `default` arm. The missing-case compiler error is the completeness gate: adding a new `DispositionAxis` constant (e.g. a sixth axis) causes a compile error at every mapping site simultaneously, preventing silent omission from the payload, structural renderer, or prose renderer. This extends the exhaustive-switch pattern established in `VocabularyTerm.axisExactMatch` (PP-20260607-7dd3ce) to all other DispositionAxis dispatch points in the codebase. The switch may return any value including `null` or `Optional.empty()` for intentionally sparse coverage — the constraint is on the switch structure, not on the return values.
