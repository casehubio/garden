---
id: PP-20260609-39c391
title: "Services in non-request-context paths must receive tenantId as an explicit parameter — never inject @RequestScoped CurrentPrincipal"
type: rule
scope: application
applies_to: "Any @ApplicationScoped service that may be called from a Quartz scheduler thread, async CDI observer (@ObservesAsync), or any other non-request-context execution path — and that needs tenantId for tenant-scoped operations (e.g. CaseMemoryStore writes, ledger writes)"
severity: important
refs:
  - docs/protocols/casehub/casememorystore-adapter-asserttenant-contract.md
violation_hint: "Service class injects CurrentPrincipal (or any @RequestScoped bean) at the field level and calls principal.tenancyId() in a method that is also invoked by a @Scheduled job or @ObservesAsync method — triggers ContextNotActiveException at runtime on the non-request thread"
created: 2026-06-09
---

Quartz scheduler threads and CDI async executor threads have no active CDI request scope. Any `@RequestScoped` bean injected as a field — including `CurrentPrincipal` — will throw `ContextNotActiveException` when accessed on these threads. Services that are legitimately called from both request-context paths (REST requests) and non-request-context paths (schedulers, async observers) must not inject `CurrentPrincipal` at all. Instead, `tenantId` must flow as an explicit `String tenantId` parameter from the caller. Each call site supplies tenantId from whatever source is available: `principal.tenancyId()` in a REST resource, an entity field after it is loaded, or a case context key retrieved from the engine. This keeps the service itself context-agnostic and allows non-request-context paths to pass the tenantId they already have without triggering CDI proxy failures.
