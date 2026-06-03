---
id: PP-20260603-fb05ef
title: "CDI classpath-presence activation requires the @ApplicationScoped implementation in a separate Maven module"
type: rule
scope: universal
applies_to: "Any Quarkus project using @DefaultBean as a fallback that should be displaced by an optional implementation"
severity: important
refs:
  - docs/protocols/universal/optional-module-pattern.md
  - docs/protocols/universal/persistence-backend-cdi-priority.md
violation_hint: "Both @DefaultBean and @ApplicationScoped implementations of the same SPI in the same Maven module — @ApplicationScoped always wins, making the @DefaultBean permanently dead code"
created: 2026-06-03
---

For classpath-presence activation to work (`@ApplicationScoped` displaces `@DefaultBean` only when the optional module is on the classpath), the `@ApplicationScoped` implementation **must live in a separate Maven module** from the `@DefaultBean` fallback. If both implementations are in the same module, the `@ApplicationScoped` bean always wins over `@DefaultBean` regardless of whether the optional dependency is present — because both are always on the classpath together. The `@DefaultBean` becomes permanently dead code and classpath-presence activation is silently defeated. Place the `@DefaultBean` fallback in the core/platform module; place the `@ApplicationScoped` real implementation in the optional module that also declares the optional library (e.g. `casehub-platform-agent-claude`) as a compile dependency. See `optional-module-pattern.md` for Jandex and POM structure requirements.
