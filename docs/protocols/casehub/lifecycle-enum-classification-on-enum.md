---
id: PP-20260620-ed1230
title: "Lifecycle enum classification methods live on the enum — consumers never enumerate states"
type: rule
scope: platform
applies_to: "Any lifecycle enum (PlanItemStatus, WorkItemStatus, CaseStatus, WorkflowStatus, CommitmentState) — applies when checking terminal, active, or any state-classification property"
severity: important
refs:
  - docs/specs/2026-06-19-lifecycle-alignment-design.md
violation_hint: "Hardcoded EnumSet, explicit == checks listing terminal states, or a static isTerminal() method on a consumer class instead of the enum"
created: 2026-06-20
---

Lifecycle classification methods (`isTerminal()`, `isActive()`) belong on the enum itself, not on consumers. Consumer code uses the enum methods; it never enumerates states explicitly unless the switch is semantically distinct per status (e.g. `ActionGateCompletionApplier` maps each status to a different gate event — that's a semantic switch, not a classification check). Adding a terminal state to an enum that owns `isTerminal()` requires updating one method; adding it to an enum without `isTerminal()` requires finding and updating every consumer that hardcodes the terminal set — the spec review for engine#539 proved this fragility when the spec itself missed `StageAutocompleteEvaluator.isTerminal()`. Established across `PlanItemStatus` (8 consumers migrated) and already present on `WorkItemStatus`.
