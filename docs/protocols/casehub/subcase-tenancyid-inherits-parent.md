---
id: PP-20260531-42fd93
title: "SubCase instances must inherit tenancyId from the parent case, never from currentPrincipal"
type: rule
scope: repo
applies_to: "casehub-engine — SubCaseExecutionHandler, any code that creates a child CaseInstance"
severity: critical
refs:
  - docs/specs/issue-299-multi-tenancy-foundation/2026-05-31-multi-tenancy-foundation-design.md
violation_hint: "child CaseInstance is saved with currentPrincipal.tenancyId() instead of parentInstance.tenancyId — findByWorkerAndType(childCaseId, SUBCASE_STARTED, tenancyId) returns empty because the SUBCASE_STARTED event is stored under the parent's tenancyId"
created: 2026-05-31
---

When SubCaseExecutionHandler creates a child CaseInstance, it must set the child's tenancyId from `parentInstance.tenancyId`, not from `currentPrincipal`. The SUBCASE_STARTED EventLog entry is written with the parent's tenancyId (the caseId on the log is the parent's). For SubCaseCompletionService to find that entry via `findByWorkerAndType(childCaseId.toString(), SUBCASE_STARTED, event.tenancyId())`, the child's tenancyId (sourced from `event.tenancyId()` on the CaseLifecycleEvent) must match the parent's tenancyId. If the child was saved with a different tenancyId — e.g. from a principal that resolved differently at creation time — the lookup silently returns empty and the parent case stalls indefinitely waiting for a completion that is never delivered.
