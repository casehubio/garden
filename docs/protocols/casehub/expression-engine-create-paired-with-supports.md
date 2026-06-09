---
id: PP-20260609-8fe136
title: "ExpressionEngine implementations that override create() must also override supportsStringCreation() to return true"
type: rule
scope: platform
applies_to: "Any ExpressionEngine CDI bean that wants to support YAML case definitions via expressionLang"
severity: important
refs:
  - casehub-engine/api/src/main/java/io/casehub/api/engine/ExpressionEngine.java
  - casehub-engine/runtime/src/main/java/io/casehub/engine/internal/engine/DefaultExpressionEngineRegistry.java
violation_hint: "Engine registers successfully and is discovered by CDI, but YAML definitions declaring expressionLang: <type> fail with UnsupportedOperationException from assertLanguageSupported() — the engine is unreachable from YAML even though it evaluates correctly at runtime"
created: 2026-06-09
---

`ExpressionEngine` provides two paired default methods: `create(String expression)` (throws `UnsupportedOperationException`) and `supportsStringCreation()` (returns `false`). An engine that overrides `create()` but leaves `supportsStringCreation()` at its default will pass runtime evaluation but be unreachable from YAML parsing: `DefaultExpressionEngineRegistry.assertLanguageSupported()` calls `engine.supportsStringCreation()` before attempting `create()` — it throws `UnsupportedOperationException` if the flag returns false, blocking any YAML case definition that declares this `expressionLang`. Always override both together: `create()` returns the evaluator; `supportsStringCreation()` returns `true` to signal YAML compatibility.
