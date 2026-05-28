---
id: PP-20260529-9f9627
title: "Strategy SPIs whose implementations may perform blocking I/O must return Uni<T>, not a synchronous type"
type: rule
scope: platform
applies_to: "All casehub repos that define strategy SPIs — engine, claudony, qhorus, and future modules"
severity: important
refs:
  - docs/protocols/casehub/spi-case-id-parameter.md
violation_hint: "SPI method returns T directly (AgentAssignment, String, etc.) instead of Uni<T>; the first implementation that makes a blocking call (HTTP, JDBC, embedding model) blocks the Vert.x IO thread at runtime under load — no compile-time signal"
created: 2026-05-29
---

Any strategy SPI method that a consumer might implement with blocking I/O (network calls, embedding services, external APIs) must return `Uni<T>` from its initial definition. The cost of declaring a synchronous SPI and then discovering blocking implementations is a breaking multi-repo API change (all callers updated, reactive chain restructured) detectable only at runtime — Vert.x IO thread blocking produces latency degradation under load, not a startup failure. The `AgentRoutingStrategy.select()` change in engine#376 was the canonical example: the sync return type was correct for in-memory trust scoring but wrong once embedding providers (engine#381) became the expected implementation. Implementations that do only in-memory work return `Uni.createFrom().item(result)`; implementations with blocking I/O use `.emitOn(Infrastructure.getDefaultWorkerPool())`. See GE-20260529-ff186e in the garden for the correct Mutiny pattern.
