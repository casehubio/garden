---
id: PP-20260605-1259d1
title: "Every module with @QuarkusTest must have quarkus-maven-plugin with generate-code goals"
type: rule
scope: platform
applies_to: "all casehub engine modules with @QuarkusTest classes"
severity: critical
refs:
  - GE-20260605-e91aa0
violation_hint: "Module has @QuarkusTest but no quarkus-maven-plugin in pom.xml — tests pass locally but hang indefinitely on 2-core CI runners"
created: 2026-06-05
---

Without `quarkus-maven-plugin` (`generate-code` + `generate-code-tests` goals), Quarkus augmentation runs inside Surefire's forked test JVM at test time. On resource-constrained CI (2-core GitHub Actions runners), augmenting a large CDI graph hangs indefinitely with no error output. The plugin moves augmentation to compile time, making it fast and predictable. Tests pass locally regardless because desktop CPUs have enough cores — the violation is only visible in CI.
