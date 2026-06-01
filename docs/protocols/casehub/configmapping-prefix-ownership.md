---
id: PP-20260601-b600ee
title: "All config keys under a @ConfigMapping prefix must be declared in the mapping interface"
type: rule
scope: platform
applies_to: "any Quarkus module that uses @ConfigMapping — including casehub-platform-scim, casehub-work, and any future module with a config interface"
severity: important
refs:
  - jvm/GE-20260529-5a8158.md
violation_hint: "SRCFG00050 at startup — a property in application.properties shares a @ConfigMapping prefix but is not declared as a method in the interface, even though another bean consumes it via @ConfigProperty"
created: 2026-06-01
---

When a `@ConfigMapping(prefix = "x.y")` interface exists, SmallRye Config treats `x.y.*` as an exclusive namespace in strict mode. Any config key under that prefix that is not declared as a method in the interface is rejected at startup with `SRCFG00050 — does not map to any root`, regardless of whether it is legitimately consumed by another CDI bean via `@ConfigProperty`. The fix is to declare the property in the mapping interface with `@WithDefault`, after which both `@ConfigProperty` injection and mapping method access coexist safely. Never add a new `@ConfigProperty` field that shares a prefix with an existing `@ConfigMapping` — always add it to the interface instead.
