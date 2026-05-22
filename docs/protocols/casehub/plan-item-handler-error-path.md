---
id: PP-20260522-1d688a
title: "Outbound handlers must fault the PlanItem on error — silent return is not acceptable"
type: rule
scope: repo
applies_to: "casehub-blackboard — all outbound handlers (SubCaseExecutionHandler, HumanTaskScheduleHandler, and any future handler that transitions a PlanItem to DELEGATED)"
severity: important
refs:
  - casehub-engine.md
  - docs/specs/2026-05-22-planitem-delegated-state-design.md
violation_hint: "Handler encounters an error (startCase throws, template not found, invalid configuration) and returns without transitioning the PlanItem — PlanItem stays in PENDING with no recovery path"
created: 2026-05-22
---

When a blackboard outbound handler fails to initiate external work — because
`startCase()` throws, the CaseDefinition is missing, a template UUID is invalid,
or configuration is malformed — it must call `faultPlanItem()` (or equivalent) to
transition the PlanItem to FAULTED. Silently returning leaves the PlanItem in PENDING
(or DELEGATED if it was already marked). `hasActivePlanItem()` returns true for
non-terminal items, so a stuck PENDING or DELEGATED PlanItem blocks the binding
from being re-scheduled on subsequent context-change events, causing the case to
hang indefinitely without any visible error. The fault transition is the
engine's signal that the binding is eligible for re-evaluation.
