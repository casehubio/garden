---
id: PP-20260522-fadf26
title: "Use markDelegated() for handlers that pass control to an external actor"
type: rule
scope: repo
applies_to: "casehub-blackboard — SubCaseExecutionHandler, HumanTaskScheduleHandler, and any future outbound handler that delegates work to an external system"
severity: important
refs:
  - casehub-engine.md
  - docs/specs/2026-05-22-planitem-delegated-state-design.md
violation_hint: "Handler calls markRunning() after dispatching to SubCase, HumanTask, or Extension — PlanItem state incorrectly signals active local computation rather than waiting for external actor"
created: 2026-05-22
---

When a blackboard handler initiates work by passing control to an external actor
(child case, human task, extension), it must transition the PlanItem to DELEGATED
via `item.markDelegated()`, not RUNNING. RUNNING is reserved for CapabilityTarget
bindings where a Quartz thread is actively executing. DELEGATED means "control has
passed to an external actor; the engine is waiting for a completion signal." The
distinction matters for LLM consumers of the normative layer and for any observer
that uses PlanItem status to reason about what the system is doing. Using RUNNING
for a waiting state is a semantic error that misleads any consumer that inspects
case state.
