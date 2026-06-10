---
id: PP-20260529-57cc3b
title: "CaseMemoryStore adapters must call MemoryPermissions.assertTenant() at the top of every operation"
type: rule
scope: platform
applies_to: "Any class implementing CaseMemoryStore, GraphCaseMemoryStore, or ReactiveCaseMemoryStore"
severity: critical
refs:
  - docs/protocols/casehub/spi-deletion-default-throws.md
  - docs/protocols/casehub/platform-spi-contract.md
violation_hint: "An adapter method that calls the backend before assertTenant() leaks memories across tenant boundaries"
created: 2026-05-29
---

Every `CaseMemoryStore` adapter — whether blocking or reactive — must call
`MemoryPermissions.assertTenant(tenantId, currentPrincipal)` as the **first statement**
of `store()`, `query()`, `erase()`, `eraseById()`, and `eraseEntity()`, before any
backend call. In reactive adapters that implement `ReactiveCaseMemoryStore` directly
(bypassing `CaseMemoryStore`), the same static utility must be called directly because
the interface default method is unreachable across interface hierarchies. Capturing
`CurrentPrincipal` before entering a `Uni` pipeline is mandatory — the `@RequestScoped`
CDI context is not guaranteed on the executor thread where the `Uni` subscription runs.
The adapter contract test suite (platform#36) mechanically verifies this on every adapter.

**Async-aware form (platform#79):** Adapters that support `@ObservesAsync` callers use the 3-arg overload `MemoryPermissions.assertTenant(tenantId, principal, requestContextActive())`. `requestContextActive()` is a private helper: `var c = Arc.container(); return c == null || c.requestContext().isActive();`. Returns `true` when (a) no CDI container (plain unit test — enforce) or (b) CDI present and request context active. Returns `false` only when CDI is present but request context is not — the `@ObservesAsync` handler condition. When `false`, the principal comparison is skipped and `tenantId` from `MemoryInput` is trusted directly. All `@QuarkusTest` adapter test classes that call adapter beans directly (not via HTTP) must be annotated `@ActivateRequestContext` to ensure `requestContextActive()` returns `true` during test execution. The security gate before capability gate ordering still applies.

**Extends to capability checks (platform#34):** When a method calls both `assertTenant()` and `requireCapability()`, `assertTenant()` must come first. Calling `requireCapability()` before `assertTenant()` leaks adapter capability information to unauthorised callers — they can probe which operations the adapter supports before being rejected for a tenant mismatch. The same applies to `graphQuery()` on `GraphCaseMemoryStore` and any future SPI extensions. Rule: security gate before capability gate, always.
