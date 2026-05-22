---
id: PP-20260523-729889
title: "casehub-engine runtime CDI requires explicit Jandex indexing and casehub-platform-expression production dep"
type: rule
scope: platform
applies_to: "all CaseHub harnesses (devtown, aml, clinical) that depend on casehub-engine"
severity: important
refs:
  - PP-20260520-5d0b91 (hitl-runtime-assembly.md)
  - PP-20260522-bbd139 (platform-cdi-index-dependency.md)
  - GE-20260523-afab1d (garden: jandex-missing-invisible-beans)
violation_hint: "quarkus:build fails with 'Unsatisfied dependency for type JQEvaluator' or similar engine SPI types, despite the class being on the classpath."
created: 2026-05-23
---

`casehub-engine` and `casehub-engine-common` ship without Jandex indices, so Quarkus ARC does not discover their CDI beans by default. Since engine#316, the engine also injects `io.casehub.platform.expression.JQEvaluator`, which lives in `casehub-platform-expression`. Every harness must: (1) add `casehub-platform-expression` as a production `pom.xml` dependency; (2) add `%prod.quarkus.index-dependency` entries for `casehub-engine` and `casehub-engine-common` in `src/main/resources/application.properties` so production augmentation discovers their beans; (3) add a plain (no `%prod.`) `quarkus.index-dependency` entry for `casehub-engine-common` in `src/test/resources/application.properties` so `@QuarkusTest` discovers `JQEvaluator`. The `%prod.` prefix prevents the full engine scan from running during `@QuarkusTest`, which would activate beans that break the Quartz scheduler. See GE-20260523-54f02a for why `exclude-types` can trigger the same unwanted scan.
