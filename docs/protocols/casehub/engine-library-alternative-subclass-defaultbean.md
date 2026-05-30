---
id: PP-20260530-8725fa
title: "Concrete engine classes extending library @Alternative beans must use @DefaultBean"
type: rule
scope: repo
applies_to: "casehub-engine-ledger and any future casehub-engine-* module that ships a concrete class extending a library @Alternative bean (e.g. JpaLedgerEntryRepository)"
severity: important
refs:
  - engine#396 (fix — CaseLedgerEntryRepository @ApplicationScoped → @DefaultBean @ApplicationScoped)
  - PP-20260514-engine-spi-noops-defaultbean (related: covers SPI no-op fallbacks)
violation_hint: "@ApplicationScoped (without @DefaultBean) on a class that extends a library @Alternative bean — when the module is added to a consumer's test classpath alongside a selected alternative, CDI resolves ambiguously between the always-active subclass and the selected alternative, corrupting startup"
created: 2026-05-30
---

Any concrete implementation class in a casehub-engine-* module that extends a library class annotated `@Alternative` MUST be annotated `@DefaultBean @ApplicationScoped`, not bare `@ApplicationScoped`. CDI does not inherit `@Alternative` from parent to child — the child's own annotations govern its CDI behaviour independently. A bare `@ApplicationScoped` subclass is always active regardless of which alternatives the consumer selects, causing CDI ambiguity when consumers add the engine module to their classpath and configure a different implementation via `quarkus.arc.selected-alternatives`. `@DefaultBean` (`io.quarkus.arc.DefaultBean`) makes the class yield automatically to any non-default qualifying bean. See `CaseLedgerEntryRepository` for the canonical example: it extends `JpaLedgerEntryRepository @Alternative` and must be `@DefaultBean @ApplicationScoped` so that consumers selecting `InMemoryLedgerEntryRepository` (or any other alternative) see no ambiguity. The `PP-20260514-engine-spi-noops-defaultbean` protocol covers the related but distinct case of SPI no-op fallbacks that yield to consumer implementations.
