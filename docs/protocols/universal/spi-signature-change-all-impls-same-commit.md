---
id: PP-20260530-88cdf9
title: "Update all in-repo SPI implementations when a method signature changes"
type: rule
scope: universal
applies_to: "Any Java project where an SPI interface has multiple implementations in the same repository (production impl, mock/no-op default, in-memory test fixture, etc.)"
severity: important
refs:
  - universal/module-tier-structure.md
violation_hint: "mvn install passes because the stale class file is reused from a previous compile; mvn clean install fails with 'does not override abstract method' on the missed implementation — the breakage is invisible until a consumer does a clean build"
created: 2026-05-30
---

When a method signature changes on an SPI interface (return type, parameter type, or name), every implementation of that interface in the same repository — including mocks, no-ops, in-memory test fixtures, and `@Alternative` CDI beans — must be updated in the same commit. Incremental Maven builds mask the breakage: `mvn install` succeeds because the stale `.class` file from the previous compile satisfies the classpath check, but `mvn clean install` fails with a compilation error on the missed implementation. The failure propagates silently to every downstream consumer on their first clean build, making root cause harder to trace. The update is always mechanical (callers get a typed member instead of a raw string, a null check becomes a null-safe field access), so the cost of updating all implementations at once is low and the cost of deferring it is high.
