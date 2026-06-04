---
id: PP-20260604-d8cc21
title: "casehub-engine-flow depends only on casehub-engine-common — not on casehub-engine runtime"
type: rule
scope: platform
applies_to: "casehub-engine-flow module and any future optional engine extension modules"
severity: important
refs:
  - casehub-engine-flow/pom.xml
  - casehub-engine-common/src/main/java/io/casehub/engine/common/spi/WorkOrchestrator.java
violation_hint: "Adding casehub-engine as a compile dependency in casehub-engine-flow pom.xml"
created: 2026-06-04
---

`casehub-engine-flow` must declare a compile-time dependency on `casehub-engine-common` only — never on `casehub-engine` (runtime). The runtime module is the CDI resolution target, not a compile-time dependency. CDI resolves `WorkOrchestrator`, `DefaultWorkOrchestrator`, and other runtime beans at startup without requiring a compile dependency. Violating this collapses the optional module boundary: adding casehub-engine-flow to the classpath would then pull in the full runtime, defeating the purpose of the isolation. SPIs that flow module code needs must live in `casehub-engine-common/spi/`; any capability requiring runtime types must be provided via CDI injection, not compile-time import.
