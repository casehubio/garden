---
id: PP-20260529-3237bd
title: "Fire CaseLifecycleEvent via .chain(completionStage()) not .invoke() in @ConsumeEvent handlers"
type: rule
scope: repo
applies_to: "casehub-engine runtime — any @ConsumeEvent handler that calls lifecycleEvents.fireAsync()"
severity: important
refs:
  - engine#393 (fix for CaseStatusChangedHandler)
  - engine#397 (tracking the remaining 5 handlers)
violation_hint: "lifecycleEvents.fireAsync(new CaseLifecycleEvent(...)) inside .invoke() — CompletionStage return value is dropped"
created: 2026-05-29
---

`@ConsumeEvent` handlers that fire CDI lifecycle events MUST use `.chain(() -> Uni.createFrom().completionStage(() -> lifecycleEvents.fireAsync(...)).onFailure().recoverWithItem(t -> { LOG.warnf(t, ...); return null; }).replaceWithVoid())` as the terminal step, not `.invoke()`. Fire-and-forget event bus publishes (`eventBus.publish()`) remain in `.invoke()` — they don't need awaiting. The Mutiny `.invoke()` callback discards the `CompletionStage` returned by `fireAsync()`, meaning `@ObservesAsync` observers may run after the handler's Uni completes and the message is considered processed. Using `.chain(completionStage())` ensures the handler's Uni does not complete until all observers have run. Failures must be recovered so observer exceptions do not break case lifecycle transitions.
