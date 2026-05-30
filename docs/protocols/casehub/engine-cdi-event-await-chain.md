---
id: PP-20260529-3237bd
title: "Fire any CDI Event.fireAsync() via .chain(completionStage()) not .invoke() in reactive handlers"
type: rule
scope: repo
applies_to: "casehub-engine runtime — any @ConsumeEvent or reactive handler that calls Event.fireAsync() (lifecycleEvents, workerDecisionEvents, or any other CDI Event)"
severity: important
refs:
  - engine#393 (fix for CaseStatusChangedHandler — pattern established)
  - engine#397 (closed — 5 remaining @ConsumeEvent handlers fixed: GoalReached, MilestoneReached, SignalReceived, CaseStarted, WorkflowExecutionCompleted; also workerDecisionEvents.fireAsync() fixed in WorkflowExecutionCompletedHandler)
  - engine#401 (code review fix — WorkerStarted fireAsync in tryProvision() missed in engine#397 batch)
violation_hint: "Event.fireAsync(...) inside .invoke() — CompletionStage return value is dropped; applies to any CDI Event, not only lifecycleEvents"
created: 2026-05-29
---

Any `Event.fireAsync()` call inside a Mutiny reactive chain MUST use `.chain(() -> Uni.createFrom().completionStage(() -> event.fireAsync(...)).onFailure().recoverWithItem(t -> { LOG.warnf(t, ...); return null; }).replaceWithVoid())`, not `.invoke()`. This applies to all CDI events fired from reactive handlers — `lifecycleEvents`, `workerDecisionEvents`, or any future `Event<T>`. Fire-and-forget event bus publishes (`eventBus.publish()`) remain in `.invoke()` — they don't need awaiting. The Mutiny `.invoke()` callback discards the `CompletionStage` returned by `fireAsync()`, meaning `@ObservesAsync` observers (including `CaseLedgerEventCapture` and `WorkerDecisionEventCapture`) may run after the handler's Uni completes and the message is considered processed. Using `.chain(completionStage())` ensures the handler's Uni does not complete until all observers have run. Failures must be recovered so observer exceptions do not break case lifecycle transitions. Note: this rule applies to both `@ConsumeEvent` handlers and any other reactive chain (e.g. `tryProvision()` in `CaseContextChangedEventHandler`) that fires a CDI event.
