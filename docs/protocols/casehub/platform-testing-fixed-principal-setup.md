---
id: PP-20260531-91b500
title: "Any @QuarkusTest module adding casehub-platform-testing must inject FixedCurrentPrincipal and call setTenancyId in @BeforeEach"
type: rule
scope: platform
applies_to: "Any @QuarkusTest class in any module that declares casehub-platform-testing as a test dependency"
severity: important
refs:
  - docs/protocols/casehub/casememorystore-adapter-asserttenant-contract.md
violation_hint: "Test fails with SecurityException from MemoryPermissions.assertTenant() or similar tenant guard, even though casehub.tenancy.default-id is set correctly in application.properties"
created: 2026-05-31
---

Adding `casehub-platform-testing` to a module's test classpath activates `FixedCurrentPrincipal
@Alternative @Priority(1)`, which silently displaces `MockCurrentPrincipal @DefaultBean`.
`FixedCurrentPrincipal` defaults `tenancyId()` to `TenancyConstants.DEFAULT_TENANT_ID` (a UUID constant),
not to the value of `casehub.tenancy.default-id` in `application.properties`. Every `@QuarkusTest` class
that invokes any SPI method guarded by `MemoryPermissions.assertTenant()` (or any equivalent tenancy check)
must inject `FixedCurrentPrincipal` and call `principal.setTenancyId(TENANT)` in a `@BeforeEach` method.
