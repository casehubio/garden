---
id: PP-20260523-729889
title: "casehub-engine runtime CDI requires Jandex indexing, casehub-platform-expression, and casehub-engine-persistence-hibernate in production"
type: rule
scope: platform
applies_to: "all CaseHub harnesses (devtown, aml, clinical) that depend on casehub-engine"
severity: important
refs:
  - PP-20260520-5d0b91 (hitl-runtime-assembly.md)
  - PP-20260522-bbd139 (platform-cdi-index-dependency.md)
  - GE-20260523-afab1d (garden: jandex-missing-invisible-beans)
violation_hint: "quarkus:build fails with 'Unsatisfied dependency for type JQEvaluator', 'CaseInstanceRepository', 'EventLogRepository', or similar engine SPI types, despite the class being on the classpath."
created: 2026-05-23
updated: 2026-05-24
---

`casehub-engine` and `casehub-engine-common` ship without Jandex indices, so Quarkus ARC does not discover their CDI beans by default. Since engine#316, the engine also injects `io.casehub.platform.expression.JQEvaluator`, which lives in `casehub-platform-expression`. Every harness must: (1) add `casehub-platform-expression` as a production `pom.xml` dependency; (2) add `casehub-engine-persistence-hibernate` as a production `pom.xml` dependency — this provides JPA implementations for `CaseInstanceRepository`, `EventLogRepository`, `SubCaseGroupRepository`, and `CaseMetaModelRepository`; `casehub-engine-persistence-memory` is test-scope only and these SPIs have no implementations in production without the hibernate module; (3) add `%prod.quarkus.index-dependency` entries for `casehub-engine`, `casehub-engine-common`, and `casehub-engine-persistence-hibernate` in `src/main/resources/application.properties` so production augmentation discovers their beans; (4) add a plain (no `%prod.`) `quarkus.index-dependency` entry for `casehub-engine-common` in `src/test/resources/application.properties` so `@QuarkusTest` discovers `JQEvaluator`. The `%prod.` prefix prevents the full engine scan from running during `@QuarkusTest`, which would activate beans that break the Quartz scheduler. See GE-20260523-54f02a for why `exclude-types` can trigger the same unwanted scan.
