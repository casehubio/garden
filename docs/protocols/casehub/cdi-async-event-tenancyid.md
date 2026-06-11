---
id: PP-20260611-d4e5cf
title: "CDI event records consumed by @ObservesAsync handlers must carry tenancyId as a field"
type: rule
scope: platform
applies_to: "Any CDI event record type that will be observed via @ObservesAsync, where the handler needs tenant-scoped operations (repository saves, context lookups, ledger writes)"
severity: important
refs:
  - docs/protocols/casehub/harness-tenantid-explicit-parameter.md
violation_hint: "Observer calls CrossTenantCaseInstanceRepository or CrossTenantEventLogRepository inside @ObservesAsync to resolve tenancyId — cross-tenant repository used outside its documented startup-recovery contract"
created: 2026-06-11
---

`@ObservesAsync` handlers execute on CDI async executor threads where no request scope is active. An observer that needs to perform tenant-scoped operations (repository writes, context updates) cannot inject `CurrentPrincipal` and cannot look up tenancyId at runtime. The tenancyId must be present on the event record itself. This extends PP-20260609-39c391 (which covers `@ApplicationScoped` services) to event record types: any record that crosses an async CDI boundary must carry `tenancyId` as a named field. Callers that fire the event already have tenancyId available — from `CaseInstance.tenancyId`, `CaseLifecycleEvent.tenancyId()`, or a similar source — and must populate it. `PlanItemCompletedEvent` and `SubCaseExecutionCompleted` are the reference implementations.
