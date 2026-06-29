---
id: PP-20260629-31cb89
title: "Each persistence backend owns its own retention mechanism — no shared retention SPI"
type: rule
scope: platform
applies_to: "All BridgeAuditStore implementations and future persistence SPIs with accumulating data"
severity: important
refs:
  - docs/protocols/casehub/store-owned-ttl-vs-spi-ttl.md
violation_hint: "A shared RetentionStrategy interface or cross-store retention-days config property appears in the SPI module"
created: 2026-06-29
---

Each store module owns its own data retention mechanism. JPA uses `@Scheduled` purge with `retention-days`; in-memory evicts on size via `max-size`; MongoDB would use native TTL indexes. The mechanism matches the storage backend's strengths — a `@Scheduled` DELETE is idiomatic for RDBMS; a TTL index is idiomatic for MongoDB; a bounded deque is idiomatic for in-memory. No cross-store property name consistency is imposed; no shared `RetentionStrategy` abstraction exists. Complementary to `store-owned-ttl-vs-spi-ttl` which governs where the TTL *property* lives; this protocol governs how retention is *implemented*.
