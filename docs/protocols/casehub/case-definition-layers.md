---
id: PP-20260518-case-definition-layers
title: "Three-layer case definition architecture — YAML and fluent Java DSL as paired authoring paths"
type: rule
scope: platform
applies_to: "casehub-engine (owns the layers); all CaseHub domain applications (devtown, aml, clinical, life, QuarkMind) when defining CasePlanModels"
severity: required
refs:
  - CNCF Serverless Workflow 1.0 specification
  - quarkus-flow (Quarkus implementation of Serverless Workflow)
  - casehub-engine-api: CaseDefinitionYamlMapper, YamlCaseHub, CaseDefinition
created: 2026-05-18
revised: 2026-05-30
---

# Protocol: Three-Layer Case Definition Architecture

**Applies to:** casehub-engine (owns the implementation); all CaseHub harnesses (devtown, aml, clinical, life, QuarkMind) when writing CasePlanModels
**Severity:** Required — collapsing layers or bypassing the mapper creates untestable or unserializable case definitions

---

## The Three Layers

This architecture is inherited from CNCF Serverless Workflow 1.0 and its Quarkus implementation (quarkus-flow). Do not reinvent it or collapse layers.

### Layer 1 — YAML

The serialized, human-readable case definition. Configurable per-deployment (or per-repo) without code changes or redeployment. Lives on the classpath as a resource file.

```yaml
name: pr-review
namespace: devtown
version: "1.0.0"
spec:
  goals:
    - name: pr-approved
      condition: ".reviews | length >= 2"
  bindings:
    - name: initial-analysis
      on: { contextChange: {} }
      when: ".pr != null and .codeAnalysis == null"
      capability: "code-analysis"
```

All expressions in YAML are **strings** evaluated by the declared expression language. The case definition declares its expression language at the top level (default: `jq`, following SW 1.0's `expressionLang` field). The mapper must use a pluggable `ExpressionEvaluatorFactory` — never hardcode `new JQExpressionEvaluator(string)`. This keeps the YAML format open to other expression languages without changing the canonical model or the mapper's callers.

### Layer 2 — Generated Schema Model (`io.casehub.model.*`)

Java classes generated from the JSON Schema (`CaseDefinition.yaml` schema file). Intermediate representation produced by Jackson YAML deserialization. Consumers never construct or hold these directly — they are an implementation detail of the mapper.

### Layer 3 — Canonical API Model (`io.casehub.api.model.CaseDefinition`)

The in-memory representation the engine operates on. Built from Layer 2 via `CaseDefinitionYamlMapper`, or built directly via the fluent Java DSL. This is the only model the engine, blackboard, and binding evaluator see.

---

## Two Authoring Paths — Paired and Equal

YAML and the fluent Java DSL are **two equal production-grade authoring paths** that both produce the same canonical `CaseDefinition` model. They are not "runtime vs test" — they are parallel user experiences. Every case definition authored in YAML MUST have a companion fluent Java DSL builder class, and vice versa.

```
+-----------------------+     +-----------------------+
|  YAML resource file   |     |  Fluent Java DSL      |
|  (declarative)        |     |  (programmatic)       |
+-----------+-----------+     +-----------+-----------+
            | CaseDefinitionYamlMapper    | direct builder calls
            v                             v
     +------------------------------------------+
     |  CaseDefinition (canonical model)        |
     |  -- the only model the engine sees       |
     +------------------------------------------+
```

### The Subset Constraint

```
YAML-expressible < Fluent Java DSL-expressible
```

All YAML case definitions can be expressed using the fluent DSL. The reverse is not true: any case definition using `LambdaExpressionEvaluator` cannot round-trip to YAML. This is a feature of the DSL path — it can express things YAML cannot (Java predicates, type-safe conditions).

### The Pairing Rule

Every case definition that uses YAML and has domain logic MUST have a `*CaseDescriptor` companion POJO carrying the business logic (worker lambdas, capability routing, SLA policies, template categories). The descriptor is the authoritative home of the logic; YAML is the authoritative source of structure (bindings, goals, plan items). Reference implementation: casehub-life#27.

**`*CaseDefinitions` structural companions** (fluent Java DSL mirroring the YAML structure) were the prior pattern. They are superseded for new harnesses — you do not need to create `*CaseDefinitions` classes for case definitions written after casehub-life#27. Existing ones remain valid for structural equivalence testing (`*EquivalenceTest`) but must not carry worker implementation logic.

Every case definition that uses the fluent DSL SHOULD have a companion YAML (unless it requires `LambdaExpressionEvaluator` — which has no YAML equivalent).

**Exception:** Pure config files and data files (application.properties, feature flags, static data) do not need descriptor companions.

---

## When to Use Which Path

Both paths are valid for production and tests. The choice depends on the authoring context:

### YAML — preferred for application case definitions

Use YAML when the case definition is **part of the deployed application** and benefits from:
- Readability by non-Java reviewers (ops, domain experts, compliance auditors)
- Configurability without recompilation or redeployment
- Declarative expression of workflow structure separate from worker logic
- Clear separation between "what the workflow does" (YAML) and "how workers execute" (Java)

YAML is the natural choice for application-level case definitions: the workflow structure is stable, reviewed as a standalone artefact, and potentially configured per-deployment.

**Runtime entry point:** extend `YamlCaseHub`:

```java
@ApplicationScoped
public class PrReviewCaseHub extends YamlCaseHub {
    public PrReviewCaseHub() {
        super("devtown/pr-review.yaml");
    }
}
```

### Fluent Java DSL — preferred for tests

Use the fluent DSL when the case definition is **co-located with the code that exercises it** and benefits from:
- Co-location with test assertions for easy review — the definition and the expectations are in the same file
- Type-safe refactoring — rename a goal or binding and the compiler catches every reference
- `LambdaExpressionEvaluator` for binding conditions — avoids JQ evaluation overhead and makes test failures readable
- Rapid iteration — change the definition and the test in one edit cycle

The test preference is strong: co-location and type safety matter most when definitions change frequently and need fast review cycles. But this preference does not make the DSL "testing only" — it is equally valid for production case definitions authored by developers who prefer programmatic construction.

```java
CaseDefinition.builder()
    .namespace("devtown")
    .name("pr-review")
    .version("1.0.0")
    .goal(Goal.builder()
        .name("pr-approved")
        .condition(ctx -> ctx.getList("reviews").size() >= 2)
        .kind(GoalKind.SUCCESS)
        .build())
    .build();
```

### Summary

| Context | Preferred path | Why |
|---------|---------------|-----|
| Application case definition (deployed workflow) | YAML + `YamlCaseHub` + `*CaseDescriptor` | YAML = structure; descriptor = business logic; both tested independently |
| Test case definition | Fluent Java DSL | Co-located, type-safe, fast iteration, lambda conditions |
| Case needing `LambdaExpressionEvaluator` | Fluent Java DSL only | Lambdas cannot be expressed in YAML |
| Developer preference for programmatic authoring | Fluent Java DSL | Equally valid — both paths produce the same model |
| Structural equivalence test (legacy) | `*CaseDefinitions` + `*EquivalenceTest` | Verify DSL companion matches YAML — optional for new harnesses; superseded by descriptor pattern |

---

## Rules

1. **Every YAML case definition with domain logic must have a `*CaseDescriptor` companion POJO.** The descriptor carries all business logic for a case type (worker lambdas, capability routing, SLA policies, template categories) and is tested independently of Quarkus. YAML is the authoritative source of structure (bindings, goals, plan items); the descriptor is the authoritative source of logic. Reference implementation: casehub-life#27.

   **`*CaseDefinitions` structural DSL companions** are superseded for new harnesses — do not create them for case definitions written after casehub-life#27 ships. Existing ones are valid as structural equivalence test companions but must not carry worker implementation logic.

2. **Declare the expression language at the case definition level.** Follow SW 1.0's `expressionLang` field. Default is `jq`. The mapper reads this field and passes it to the `ExpressionEvaluatorFactory` — no hardcoded evaluator type.

3. **Do not hardcode `new JQExpressionEvaluator(string)` in `CaseDefinitionYamlMapper`.** Use an `ExpressionEvaluatorFactory` so the mapper is expression-language-agnostic. This is the engine gap tracked in casehubio/engine#280 (open).

4. **Do not bypass `CaseDefinitionYamlMapper`.** It is the single conversion point from YAML to the canonical model. Custom parsers or direct Jackson deserialization to `io.casehub.api.model.*` will break as the schema evolves.

5. **Do not hold `io.casehub.model.*` types outside the mapper.** Generated schema models are an implementation detail. Inject or pass `CaseDefinition` (Layer 3), not schema model objects.

6. **Do not use `LambdaExpressionEvaluator` in YAML-loaded definitions.** It cannot be expressed in YAML and will not survive serialization. Lambda conditions belong in the DSL companion or in tests.

7. **Do not collapse YAML and canonical model into a single type.** The separation exists so that YAML format can evolve (via schema versioning) independently of the in-memory API.

---

## Violation Hints

- A YAML case definition with domain logic and no `*CaseDescriptor` companion
- Worker lambdas placed in a `*CaseDefinitions` FuncDSL companion rather than a `*CaseDescriptor` POJO
- Custom Jackson deserialization producing `io.casehub.api.model.*` directly (bypasses mapper)
- `io.casehub.model.*` types leaking into service or handler code
- `LambdaExpressionEvaluator` in a case definition that is registered via YAML at startup
- A harness loading YAML without `YamlCaseHub` or `CaseDefinitionYamlMapper`
