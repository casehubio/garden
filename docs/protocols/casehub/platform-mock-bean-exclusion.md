---
id: PP-20260601-70e9ea
title: "Exclude casehub-platform @DefaultBean mock SPIs in modules that also index casehub-persistence-memory"
type: rule
scope: platform
applies_to: "Any casehub engine module whose test classpath includes both casehub-platform (compile dep) and casehub-persistence-memory (indexed via quarkus.index-dependency)"
severity: important
refs:
  - blackboard/src/test/resources/application.properties
  - resilience/src/test/resources/application.properties
  - work-adapter/src/test/resources/application.properties
violation_hint: "CDI AmbiguousResolutionException at test startup — 'Ambiguous dependencies for type CurrentPrincipal/GroupMembershipProvider/PreferenceProvider and qualifiers [@Default]'"
created: 2026-06-01
---

`casehub-platform` ships three `@DefaultBean @ApplicationScoped` mock SPI implementations
(`MockCurrentPrincipal`, `MockGroupMembershipProvider`, `MockPreferenceProvider`) that return
null or uninitialised values. `casehub-persistence-memory` ships `DefaultTestPrincipal`
(`@DefaultBean`), which returns `TenancyConstants.DEFAULT_TENANT_ID`. When both libraries
land on the same test CDI classpath, Quarkus ARC cannot resolve the ambiguity between two
`@DefaultBean` implementations of the same SPI — it reports **Ambiguous dependencies**, not
Unsatisfied, and refuses to start. Any module that depends on `casehub-platform` AND indexes
`casehub-persistence-memory` MUST exclude all three platform mocks in its test
`application.properties` via `quarkus.arc.exclude-types`:

```properties
quarkus.arc.exclude-types=...,\
  io.casehub.platform.mock.MockCurrentPrincipal,\
  io.casehub.platform.mock.MockGroupMembershipProvider,\
  io.casehub.platform.mock.MockPreferenceProvider
```

`DefaultTestPrincipal` returns the sentinel and must be the surviving `@DefaultBean` for all
engine integration tests. Any new module that adds `casehub-platform` as a compile dependency
must apply this exclusion at the same time.
