---
id: PP-20260610-ecc2b2
title: "CASE_STARTED EventLog payloads must use panel document format after the panels migration"
type: rule
scope: repo
applies_to: "Any test code that manually constructs CASE_STARTED EventLog entries (e.g. SubCaseOutputMappingRecoveryTest, recovery integration tests)"
severity: critical
refs:
  - runtime/src/main/java/io/casehub/engine/internal/engine/handler/CaseStartedEventHandler.java
  - runtime/src/main/java/io/casehub/engine/internal/engine/recovery/DefaultWorkerExecutionRecoveryService.java
violation_hint: "IllegalArgumentException: Cannot construct instance of LinkedHashMap from String value 'ORDER-1' — fromPanelDocument() is interpreting a flat map key's value (a String) as panel data (expected Map)"
created: 2026-06-10
---

After the panels migration (engine#80/#81), `CaseStartedEventHandler` writes `instance.getCaseContext().asJsonNode()` as the `CASE_STARTED` payload. This is a panel document: `{"working":{...},"semantic":{},"episodic":{}}`. `DefaultWorkerExecutionRecoveryService.rebuildStateContext()` reads it via `CaseContextImpl.fromPanelDocument()`, which expects panel-name keys at the top level. Test code that manually writes `CASE_STARTED` events with flat payloads (`{"orderId":"X"}`) will fail at recovery time with an IllegalArgumentException. Fix: write the payload as `Map.of("working", Map.of("orderId", "X"), "semantic", Map.of(), "episodic", Map.of())`.
