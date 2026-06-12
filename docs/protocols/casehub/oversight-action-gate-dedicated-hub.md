---
id: PP-20260612-181367
title: "Oversight action gates use a dedicated YamlCaseHub — never programmatic bindings on an existing case definition"
type: rule
scope: application
applies_to: "Any casehub harness (aml, clinical, devtown, life) that implements an ActionRiskClassifier gate for consequential agent actions"
severity: important
refs:
  - casehub/HARNESS-INDEX.md
violation_hint: "ClinicalAdverseEventCaseHub.getDefinition() adds a Binding and Worker programmatically — the binding fires on the initial empty-context event, the plan item goes RUNNING before the full context arrives, and all subsequent context events cannot re-dispatch it."
garden_ref: "GE-20260612-9ff1c6"
created: 2026-06-12
---

Oversight action gates (for `ActionRiskClassifier`-gated workers) must be implemented as a dedicated `YamlCaseHub` subclass with its own YAML case definition — following the AML Layer 9 pattern (`AmlOversightCaseHub`). Do NOT add programmatic `ContextChangeTrigger` worker bindings to an existing `YamlCaseHub` subclass.

The engine fires `CaseContextChangedEvent` before the initial context is applied to the live `CaseInstance`. A programmatic binding with a null-equality filter (e.g. `.susarAssessmentComplete == null`) fires vacuously on the initial empty-context event. `PlanningStrategyLoopControl.filterToDispatchable()` marks the plan item RUNNING immediately, permanently blocking re-dispatch when the full context arrives. The worker receives all-null `inputData`. YAML `humanTask` bindings in the same case are unaffected because their filters require affirmative Boolean fields that are false against the empty context.
