---
id: PP-20260602-84e308
title: "Adopt casehub-platform implementations progressively — mock by default, real when the concern is production-ready"
type: principle
scope: application
applies_to: "any repo consuming casehub-platform — decisions about which SPI implementation modules to include"
severity: guidance
refs:
  - docs/protocols/casehub/platform-spi-contract.md
  - docs/protocols/casehub/platform-testing-fixed-principal-setup.md
  - docs/protocols/casehub/platform-mock-bean-exclusion.md
created: 2026-06-02
---

casehub-platform ships `@DefaultBean` mock implementations for every SPI so consumers can build without wiring real infrastructure. Add real implementations only when the concern is production-ready: switch `CurrentPrincipal` from `MockCurrentPrincipal` to `casehub-platform-oidc` when the REST layer needs real authenticated users; switch `PreferenceProvider` from `MockPreferenceProvider` to `casehub-platform-config` when scope-aware startup YAML is needed, and to `casehub-platform-persistence-jpa` or `persistence-mongodb` when runtime editing becomes a product feature; switch `GroupMembershipProvider` from the no-op `@DefaultBean` to `casehub-platform-scim` when group-based access control is a real requirement. Displacement is automatic — adding a higher-priority implementation to the classpath activates it and the mock steps aside. For `@QuarkusTest`, always use `casehub-platform-testing` (`FixedCurrentPrincipal` with mutable setters) rather than config-property overrides on `MockCurrentPrincipal`.
