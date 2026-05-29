---
id: PP-20260530-2ad9a4
title: "Observe domain CDI events, not WorkItemLifecycleEvent, for per-domain-operation triggers"
type: rule
scope: application
applies_to: "casehub harness notification listeners, side-effect observers — any @ObservesAsync handler that must fire exactly once per domain operation"
severity: important
refs:
  - casehub/HARNESS-INDEX.md
violation_hint: "@ObservesAsync WorkItemLifecycleEvent used to trigger notification or side-effect; fires multiple times when an engine case creates multiple WorkItems for one domain event (e.g. Grade 4/5 AE: senior-safety-monitor + dsmb)"
created: 2026-05-30
---

When a casehub harness needs to react exactly once to a domain event (notification, ledger entry, side-effect), observe a **domain CDI event** fired by the service layer (`AdverseEventReportedEvent`, `ProtocolDeviationResolvedEvent`) rather than `WorkItemLifecycleEvent` from casehub-work. Engine cases can create multiple WorkItems per domain event — one per humanTask binding — so `WorkItemLifecycleEvent` fires N times per operation. Domain CDI events are fired once, at the service boundary, before engine orchestration begins; they carry the domain entity ID and context directly. This is the pattern established by `SafetyOfficerNotificationListener` (casehubio/clinical#11) and `SponsorNotificationListener`.
