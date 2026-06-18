---
id: PP-20260618-priv-no-async
title: "Privileged administrative operations must not bypass their security gate in async context"
type: rule
scope: platform
applies_to: "Any MemoryPermissions check added for privileged operations (not unauthenticated async-hop operations)"
severity: important
refs:
  - docs/protocols/casehub/casememorystore-adapter-asserttenant-contract.md
created: 2026-06-18
---

The 3-arg `assertTenant(tenantId, principal, requestContextActive)` form exists to accommodate
`@ObservesAsync` callers — when `requestContextActive=false`, the principal comparison is skipped
because the CDI request scope is not available on the async thread; the tenantId from the domain
object is trusted directly. This bypass applies ONLY to authentication (tenant validation) in
fire-and-forget async handlers.

Privileged operations — such as `assertCrossTenantAdmin(principal)` — must NOT have an async bypass
form. Cross-tenant erasure is a deliberate administrative action, never initiated from `@ObservesAsync`
context. Adding an async bypass to a privilege check creates a security hole: any `@ObservesAsync`
handler could call the method and bypass the admin requirement.

The rule: if a new `MemoryPermissions` check guards a privilege (not just tenant identity), it uses
only a 1-arg form. No requestContextActive variant is added. Document that it must not be called from
`@ObservesAsync`.
