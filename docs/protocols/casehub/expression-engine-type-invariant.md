---
id: PP-20260609-3c86d1
title: "expressionLang, ExpressionEvaluator.type(), and ExpressionEngine.type() must be identical"
type: rule
scope: platform
applies_to: "Any class implementing ExpressionEngine and overriding create()"
severity: critical
refs:
  - casehub-engine/api/src/main/java/io/casehub/api/engine/ExpressionEngine.java
  - casehub-engine/runtime/src/main/java/io/casehub/engine/internal/engine/DefaultExpressionEngineRegistry.java
violation_hint: "ExpressionEngine.create() returns an evaluator whose type() does not match the engine's type() — causes silent misrouting at evaluation time; expressions are evaluated by the wrong engine"
created: 2026-06-09
---

The three-way identity `expressionLang (YAML) == ExpressionEvaluator.type() (returned by create()) == ExpressionEngine.type()` is load-bearing for the entire pluggable expression evaluation chain. `CaseDefinitionYamlMapper` reads `expressionLang` from the case definition YAML, calls `ExpressionEngineRegistry.create(expression, expressionLang)`, which dispatches to the matching engine's `create()` — returning an evaluator. At runtime, `DefaultExpressionEngineRegistry.evaluate()` dispatches again by `evaluator.type()`. If any of the three values differ, an expression created for one language is silently evaluated by a different engine. `DefaultExpressionEngineRegistry.create()` asserts this invariant at runtime: `evaluator.type().equals(expressionLang)` — violations throw `IllegalStateException` immediately on first use.
