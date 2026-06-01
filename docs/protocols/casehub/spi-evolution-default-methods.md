---
id: PP-20260601-81b9e5
title: "Add new optional SPI capabilities as Java interface default methods with safe no-op returns"
type: rule
scope: platform
applies_to: "All casehub SPIs with multiple implementations — WorkerExecutionManager, CommitmentStore, WorkItemStore, PlanItemStore, and any future platform SPI"
severity: important
refs:
  - docs/protocols/casehub/engine-spi-noops-defaultbean.md
  - docs/protocols/casehub/platform-spi-contract.md
  - docs/protocols/universal/spi-default-method-contract-test.md
violation_hint: "A new SPI method is added without a default implementation, breaking all existing implementations in a coordinated multi-repo API change; or the method is added without a matching abstract contract test case, allowing JPA and in-memory implementations to diverge semantically"
created: 2026-06-01
---

When adding a new optional capability to an existing casehub SPI, declare it as a Java `interface default` method with a safe no-op return — `List.of()` for collections, `Optional.empty()` for lookups, `0` for counts. Only implementations that can meaningfully fulfil the capability override it; all others get the safe no-op for free. The corollary: every new SPI method — default or not — must also be added to the module's abstract contract test (e.g. `CommitmentStoreContractTest`, `PlanItemStoreContractTest`). The contract test is the only mechanism that verifies all implementations — JPA, in-memory, reactive — honour the same semantics; adding a method without a contract case allows implementations to diverge silently. This pattern covers Java `interface default` for optional capabilities and is distinct from CDI `@DefaultBean` for no-op SPI bean registration — see `engine-spi-noops-defaultbean.md`.
