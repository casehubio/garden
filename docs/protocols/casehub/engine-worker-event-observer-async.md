---
id: PP-20260531-70879a
title: "Casehub harness observers of engine worker events must use @ObservesAsync"
type: rule
scope: application
applies_to: "any casehub harness (aml, clinical, devtown, etc.) that observes WorkerDecisionEvent or other engine-fired CDI events"
severity: important
refs:
  - casehub-engine/GE-20260531-864d8e (garden: @Observes silently never fires for WorkerDecisionEvent)
  - casehub/engine-cdi-event-await-chain.md (engine-side: fireAsync must be awaited in reactive chains)
violation_hint: "Observer method annotated with @Observes WorkerDecisionEvent is never called — no error, no log, zero invocations"
created: 2026-05-31
---

Any observer of `WorkerDecisionEvent` (or any other event fired by casehub-engine via `Event.fireAsync()`) must use `@ObservesAsync`, not `@Observes`. The engine fires worker events asynchronously; synchronous observers registered with `@Observes` are silently skipped for async events with no error or warning. Use `@Transactional(TxType.REQUIRES_NEW)` on the `@ObservesAsync` method when the handler needs database access — this gives it an independent transaction decoupled from the engine worker's transaction. Note: `engine-cdi-event-await-chain.md` covers the engine-side obligation (awaiting the `CompletionStage`); this protocol covers the consumer-side obligation.
