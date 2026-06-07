---
id: PP-20260605-e63850
title: "CaseMemoryStore storeAll() must guarantee atomicity — single-transaction for JDBC, pre-flight assertTenant for REST"
type: rule
scope: platform
applies_to: "All CaseMemoryStore adapter implementations — memory-jpa, memory-sqlite, memory-inmem, memory-mem0, and any future adapters (memory-graphiti, etc.)"
severity: important
refs:
  - platform-api/src/main/java/io/casehub/platform/api/memory/CaseMemoryStore.java
  - docs/superpowers/specs/2026-06-05-memory-cdi-priority-and-emission-design.md
violation_hint: "JDBC: storeAll() calls assertTenant() only on item 0 or uses N separate @Transactional calls — commits items before violation detected. REST: storeAll() fires HTTP before pre-flighting all inputs — stores item 0 before item 1's tenant check fires"
created: 2026-06-05
---

Any CaseMemoryStore adapter overriding `storeAll()` must wrap all N inserts in a single
`@Transactional(REQUIRED)` scope, call `MemoryPermissions.assertTenant()` per item inside
the batch (not just on the first item), and propagate `SecurityException` for any mismatch.
This guarantees atomicity: no entries are persisted if any item fails the tenant check.
The SPI default (`store() × N`) must not be used when partial-write safety is required —
it issues a separate transaction per call, committing earlier items before the violation
is detected. Exception type on mismatch must be `SecurityException` (from `assertTenant`),
not `IllegalArgumentException`, so callers can write adapter-neutral catch clauses.

**REST-backed adapters** (Mem0, Graphiti, and any future HTTP-delegating adapters) have no
JPA transaction. The equivalent guarantee is: pre-flight `MemoryPermissions.assertTenant()`
for **all** inputs before issuing any HTTP request, then fail-fast on the first HTTP error.
A `SecurityException` from `storeAll()` is therefore always clean — nothing was sent to the
backend. Items already persisted before a mid-batch HTTP failure cannot be rolled back — this
is a documented REST adapter limitation and must be noted in the adapter's Javadoc or spec.
