---
id: PP-20260522-bbd139
title: "casehub-platform CDI modules require quarkus.index-dependency in every consuming test config"
type: rule
scope: platform
applies_to: "any engine or consumer module with @QuarkusTest that injects a bean from a casehub-platform artifact"
severity: important
refs:
  - docs/protocols/casehub/FOUNDATION-INDEX.md
violation_hint: "UnsatisfiedResolutionException for a platform type at @QuarkusTest startup, even though the class compiles fine"
created: 2026-05-22
---

Quarkus only auto-indexes the application module for CDI bean discovery; library JARs on the classpath are not scanned unless explicitly listed. Any `casehub-platform-*` artifact that contains `@ApplicationScoped` beans must be added to `quarkus.index-dependency` in every consuming module's `src/test/resources/application.properties` that runs `@QuarkusTest`. The key name is arbitrary (use the artifact-id slug); the group-id is always `io.casehub`. This applies to `casehub-platform-expression` (for `JQEvaluator`) and `casehub-platform` (for `MockPreferenceProvider`, required by casehub-work's `ExpiryLifecycleService` and `RoutingCursorCleanupJob` since work#218), and any future platform artifacts that ship CDI beans. Both must also be production `pom.xml` dependencies — the test index-dependency alone is insufficient. The symptom is always a clean compile followed by a startup-time `UnsatisfiedResolutionException`.
