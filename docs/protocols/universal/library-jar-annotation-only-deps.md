---
id: PP-20260604-88f660
title: "Library JARs (no quarkus:build) must depend on annotation-only libraries, not Quarkus extensions"
type: rule
scope: universal
applies_to: "Any Maven module with packaging=jar and no quarkus-maven-plugin build goal — i.e. library modules consumed as JARs by Quarkus applications"
severity: important
refs:
  - docs/protocols/universal/optional-module-pattern.md
violation_hint: "A library module declares quarkus-micrometer, quarkus-cache, or any quarkus-* extension as a compile-scope dependency; the consuming application's augmentation phase then encounters conflicting deployment metadata and may fail or behave unexpectedly"
created: 2026-06-04
---

When a library JAR module (packaging=jar, no `<goal>build</goal>` in the quarkus-maven-plugin)
needs annotations that require Quarkus ARC interception — such as `@Timed` (Micrometer),
`@CacheResult` (SmallRye Cache), or `@Retry` (MicroProfile Fault Tolerance) — depend on the
**annotation-only** library at compile scope, NOT the Quarkus extension.

**Correct pattern for `@Timed`:**
```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-core</artifactId>
    <!-- Provides @Timed annotation. Interception wired by the consuming app's quarkus-micrometer. -->
</dependency>
```

**Violation (do not do):**
```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-micrometer</artifactId>  <!-- Extension — NOT for library JARs -->
</dependency>
```

**Why:** Quarkus extension JARs carry two artifacts — a `runtime` JAR and a `deployment` JAR.
The `deployment` artifact contains `BuildStep` classes and augmentation metadata. When a
library module carries these artifacts as compile-scope dependencies, the consuming application's
augmentation phase encounters unexpected deployment contributors that were not designed to run in
that context — causing confusing failures or silently incorrect CDI wiring.

The ARC interceptor for `@Timed` (or any other annotation) is registered by the consuming
application's own extension (e.g. `quarkus-micrometer`) at its augmentation time. The library
module only needs the annotation class on its classpath — provided by the annotation-only library.
The annotation is silently ignored if the consuming application has no matching extension registered.
