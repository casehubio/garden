---
id: PP-20260522-jq-evaluation-canonical
title: "JQ evaluation must go through JQEvaluator — never hand-rolled or direct JsonQuery"
type: rule
scope: platform
applies_to: "All casehub repos writing JQ expression evaluation code"
severity: required
refs:
  - casehubio/engine#314
  - casehubio/engine#317
created: 2026-05-22
---

# Protocol: Canonical JQ Evaluation

**Applies to:** All casehub repos  
**Severity:** Required — violations produce duplicate evaluator instances, missing secret/config injection, and silent behavioural divergence

---

## The Rule

**In CDI beans:** inject `JQEvaluator` (`io.casehub.engine.internal.jq.JQEvaluator`) and call `eval()`. Never instantiate `JsonQuery` directly. Never hand-roll `{ key: .path }` template parsers.

**In non-CDI API-layer classes** (e.g. `Agent`, `AgentBuilder`): use `JqTransformer` (`io.casehub.api.model.ai.JqTransformer`). Build the `Scope` once in the constructor — never per `apply()` call.

**Never** write a new class that wraps `net.thisptr.jackson.jq.JsonQuery` independently.

---

## Why

Three independent jq evaluator implementations emerged before this protocol existed:

| Class | Problem |
|---|---|
| `JQEvaluator` (engine) | Canonical — correct scope, secret/config injection |
| `JqConditionEvaluator` (casehub-work) | Separate repo, compiled `JsonQuery` per call |
| `JqTransformer` (engine/api) | Rebuilt `Scope` + reloaded builtins on every `apply()` call |
| `CaseContextImpl.evalObjectTemplate` | Hand-rolled mini-DSL, didn't support nested objects |

Each instance diverges over time: different error handling, missing $secret/$config injection, inconsistent behaviour on edge cases (null, empty, nested objects).

---

## How to Evaluate JQ in a CDI Bean

```java
@Inject JQEvaluator jqEvaluator;

private static final TypeReference<Map<String, Object>> MAP_TYPE = new TypeReference<>() {};
private static final ObjectMapper MAPPER = new ObjectMapper();

// Evaluate against a CaseContext
ValidationResult vr = jqEvaluator.eval(expression, context.asJsonNode());

// Evaluate against a raw Map
ValidationResult vr = jqEvaluator.eval(expression, MAPPER.valueToTree(data));

// Extract result as Map
if (vr.ok() && vr.output() != null && !vr.output().isEmpty()) {
    Map<String, Object> result = MAPPER.convertValue(vr.output().get(0), MAP_TYPE);
}
```

With secrets or config maps (declared in `use.secrets` / `use.configMaps`):
```java
ValidationResult vr = jqEvaluator.eval(expression, node, secretNames, configMapNames);
```

---

## Expression Evaluation is Not a Data Responsibility

`CaseContext` and similar data-holder objects must not evaluate jq expressions against themselves. Evaluation belongs in CDI beans that inject `JQEvaluator`. The mistake of placing `evalObjectTemplate` on the `CaseContext` interface was corrected in engine#314 — do not repeat it.

---

## Platform Tier Note

`JQEvaluator` currently lives in `casehub-engine-common`, making it available to all modules that depend on `casehub-engine-common` or `casehub-engine`. The long-term home is `casehub-platform` (tracked in engine#317 — once extracted, all repos including `casehub-work` can consume the canonical evaluator without rolling their own).

---

## Violation Signals

- A new class that imports `net.thisptr.jackson.jq.JsonQuery`
- A class with `{ key: .path }` string parsing logic
- `BuiltinFunctionLoader.getInstance().loadFunctions()` called outside of a `@PostConstruct` or constructor
- A method named `evalTemplate`, `evalMapping`, or similar that builds a Map by string-splitting
