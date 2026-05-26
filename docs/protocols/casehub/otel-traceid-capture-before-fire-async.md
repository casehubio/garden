---
id: PP-20260526-3a22d3
title: "Capture OTel trace ID synchronously before CDI fireAsync() — never in the async handler"
type: rule
scope: platform
applies_to: "Any casehub module that fires CaseLifecycleEvent or other CDI events via fireAsync() and needs OTel trace correlation in downstream handlers"
severity: important
refs:
  - ledger/src/main/java/io/casehub/ledger/service/CaseLedgerEventCapture.java
violation_hint: "CaseLedgerEntry.traceId (or any OTel-dependent field) is always null after async CDI delivery"
created: 2026-05-26
---

`@ObservesAsync` handlers execute on a managed executor thread where OTel's `ThreadLocal`-backed span context is absent. Capture `traceIdProvider.currentTraceId().orElse(null)` synchronously at the fire-site, before `fireAsync()`, and carry the value through the event record. Do not attempt to read the OTel context inside the `@ObservesAsync` handler — it will always be empty. The downstream enricher (e.g. `TraceIdEnricher`) must null-guard: `if (entry.traceId != null) return;` so the pre-captured value is preserved through `@PrePersist`. Applied to all `CaseLifecycleEvent` fire-sites in engine#342.
