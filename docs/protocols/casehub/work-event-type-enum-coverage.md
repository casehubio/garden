---
id: PP-20260526-25c59b
title: "Every new WorkItemService audit event string must have a matching WorkEventType enum value"
type: rule
scope: repo
applies_to: "casehub-work — WorkItemService, WorkEventType, WorkItemLifecycleEvent"
severity: important
refs:
  - runtime/src/main/java/io/casehub/work/runtime/event/WorkItemLifecycleEvent.java
  - api/src/main/java/io/casehub/work/api/WorkEventType.java
violation_hint: "New audit event string ('DEADLINE_EXTENDED', 'SIGNAL_RECEIVED', etc.) added to WorkItemService.audit() and WorkItemLifecycleEvent.of() without a matching WorkEventType constant — eventType() silently returns CREATED for the new event, breaking all CDI observers that switch on WorkEventType"
created: 2026-05-26
---

`WorkItemLifecycleEvent.eventType()` resolves the event string via `WorkEventType.valueOf(name.toUpperCase())` inside a try-catch that silently returns `WorkEventType.CREATED` on `IllegalArgumentException`. Any audit event string not present in `WorkEventType` is therefore indistinguishable from a CREATED event at the CDI observer layer — no compile-time error, no runtime warning. When adding a new lifecycle transition to `WorkItemService` (with a new audit event string in `audit()` and `WorkItemLifecycleEvent.of()`), add the matching enum constant to `WorkEventType` in the same commit.
