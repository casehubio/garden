---
id: PP-20260529-3237bd
title: "CDI Event.fireAsync() dispatch pattern depends on event purpose — audit events use .invoke(), decision events use .chain()"
type: rule
scope: repo
applies_to: "casehub-engine runtime — any @ConsumeEvent or reactive handler that calls Event.fireAsync() (lifecycleEvents, workerDecisionEvents, or any other CDI Event)"
severity: important
refs:
  - engine#393 (fix for CaseStatusChangedHandler — pattern established)
  - engine#397 (closed — 5 remaining @ConsumeEvent handlers fixed)
  - engine#401 (code review fix — WorkerStarted fireAsync in tryProvision() missed in engine#397 batch)
  - engine#491 (proved chained CDI observers block case state progression under H2 lock contention)
  - engine#493 (standardised fire-and-forget for all audit CDI events across handlers)
violation_hint: "Audit CDI event inside .chain() — gates case state progression on observer completion. Decision CDI event inside .invoke() — drops CompletionStage, outcome may not be available."
created: 2026-05-29
updated: 2026-06-16
---

CDI `Event.fireAsync()` calls in reactive handlers use one of two patterns depending on the event's purpose:

**Audit/observability events** (`CaseLifecycleEvent`, `WorkerDecisionEvent`): fire-and-forget via `.invoke()`. These must never gate case state progression — a slow observer (e.g. ledger capture doing DB writes under contention) must not stall the handler's Uni chain or delay `CONTEXT_CHANGED` dispatch.

```java
.invoke(() -> {
    lifecycleEvents.fireAsync(...)
        .whenComplete((v, t) -> {
            if (t != null) LOG.warnf(t, "CaseLifecycleEvent observer failed...");
        });
})
```

**Decision-influencing events** (events whose outcome affects the current handler's control flow): use `.chain(completionStage())` so the handler waits for the result before proceeding.

```java
.chain(() -> Uni.createFrom().completionStage(() -> event.fireAsync(...))
    .onFailure().recoverWithItem(t -> { LOG.warnf(t, ...); return null; })
    .replaceWithVoid())
```

The original rule (engine#393, #397, #401) mandated `.chain()` for all CDI events. Engine#491 proved this is wrong for audit events: chained CDI observers can block case state progression when observers perform slow I/O. Engine#493 standardised the fire-and-forget pattern across `SignalReceivedEventHandler`, `CaseStartedEventHandler`, and `WorkflowExecutionCompletedHandler`.

Fire-and-forget event bus publishes (`eventBus.publish()`) remain in `.invoke()` — they don't return a `CompletionStage`.
