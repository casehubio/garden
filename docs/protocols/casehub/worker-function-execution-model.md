---
id: PP-20260531-worker-func-exec
title: "Worker functions must use quarkus-flow workflows — YAML or Java FuncDSL, never raw lambdas"
type: rule
scope: platform
applies_to: "All CaseHub domain applications (devtown, aml, clinical, life, QuarkMind) when defining Worker functions for CasePlanModels"
severity: required
refs:
  - quarkus-flow FuncDSL (io.serverlessworkflow.fluent.func.dsl.FuncDSL)
  - quarkus-flow FuncWorkflowBuilder (io.serverlessworkflow.fluent.func.FuncWorkflowBuilder)
  - casehub-engine Worker.Builder.function() — accepts Workflow, Function, Agent, File
  - PP-20260518-case-definition-layers (three-layer case definition architecture)
created: 2026-05-31
---

# Protocol: Worker Function Execution Model

**Applies to:** All CaseHub harnesses defining `Worker` functions for case definitions
**Severity:** Required — raw lambdas bypass the workflow execution pipeline, losing tracing, composition, and backend portability

---

## Origin

This protocol is inspired by the same duality that governs quarkus-flow itself: YAML workflows and Java FuncDSL are two equal authoring paths for the same execution model. The case-definition-layers protocol (PP-20260518) applies this principle to case definitions (YAML + fluent DSL). This protocol applies it to the worker functions within those case definitions.

The pairing principle flows down from quarkus-flow → case definitions → worker workflows.

---

## The Rule

Worker functions in production case definitions MUST use quarkus-flow workflows. Two authoring paths are available — choose based on context:

1. **Java FuncDSL** — `FuncWorkflowBuilder.workflow().tasks(...).build()` with `FuncDSL` task composition
2. **YAML workflow** — declarative workflow definition loaded from a classpath resource

Do not pass a raw `Function<Map<String, Object>, Map<String, Object>>` lambda to `Worker.Builder.function()`.

---

## Two Authoring Paths — Paired and Equal

Just as case definitions have YAML and fluent DSL as paired authoring paths, worker workflows have **YAML workflows** and **Java FuncDSL** as paired authoring paths. Both produce a `Workflow` that the engine executes through the quarkus-flow pipeline.

```
+----------------------------+     +----------------------------+
|  YAML workflow resource    |     |  Java FuncDSL              |
|  (declarative)             |     |  (programmatic)            |
+-------------+--------------+     +-------------+--------------+
              | workflow loader                   | workflow().tasks(...).build()
              v                                   v
     +------------------------------------------+
     |  Workflow (quarkus-flow execution model)  |
     |  — tracing, I/O mapping, composition     |
     +------------------------------------------+
```

### The Subset Constraint

```
YAML workflow-expressible < Java FuncDSL-expressible
```

All YAML workflows can be expressed using the Java FuncDSL. The reverse is not true: any workflow using Java lambdas (CDI service calls, in-process computation) cannot be expressed in YAML. This mirrors the case definition subset constraint.

### The Pairing Rule

Worker workflows that use YAML SHOULD have a companion Java FuncDSL equivalent (unless the workflow is trivially a single HTTP call). Worker workflows that use Java FuncDSL SHOULD have a companion YAML (unless they require lambdas — which has no YAML equivalent).

Workers that call CDI services (ledger writes, commitment creation, channel dispatch) require lambdas and therefore can only use Java FuncDSL. These workers have no YAML companion — the pairing rule does not apply to lambda-dependent workers.

---

## When to Use Which Path

| Worker needs to... | Path | Why |
|--------------------|------|-----|
| Call CDI services (JPA, ledger, qhorus) | Java FuncDSL only | Requires lambdas — no YAML equivalent |
| Pure data transformation / mapping | Either — prefer YAML | Declarative, no Java logic needed |
| HTTP call to external service | Either — prefer YAML | `get(name, url)` in FuncDSL, equivalent in YAML |
| LLM/AI agent invocation | Java FuncDSL | `agent(ref::method, Type.class)` — lambda-based |
| Multi-step composition (fetch then process) | Either — FuncDSL for mixed steps | YAML if all steps are declarative; FuncDSL if any step needs a lambda |

---

## Java FuncDSL — Correct Usage

```java
import static io.serverlessworkflow.fluent.func.FuncWorkflowBuilder.workflow;
import static io.serverlessworkflow.fluent.func.dsl.FuncDSL.function;

Worker.builder()
    .name("destination-researcher")
    .capabilities(researchCap)
    .function(
        workflow("destination-research")
            .tasks(
                function(s -> {
                    Map<String, Object> ctx = (Map<String, Object>) s;
                    String destination = (String) ctx.get("destination");
                    return Map.of("options", List.of(...), "estimatedCost", 2500);
                }, Map.class))
            .build())
    .build();
```

### Incorrect — Raw Lambda

```java
Worker.builder()
    .name("destination-researcher")
    .capabilities(researchCap)
    .function((Map<String, Object> input) -> {
        // Raw lambda — bypasses workflow pipeline
        return Map.of("options", List.of(...), "estimatedCost", 2500);
    })
    .build();
```

---

## Why Raw Lambdas Are Wrong

`Worker.Builder.function()` accepts four types:

| Type | Import | What it is |
|------|--------|------------|
| `Workflow` | `FuncWorkflowBuilder.workflow()` | Composed quarkus-flow pipeline — the standard execution model |
| `Function<Map, Map>` | raw lambda | Bare function — no pipeline, no tracing, no composition |
| `Agent` | `FuncDSL.agent()` | LLM/AI agent task — structured agent invocation |
| `File` | file reference | File-based workflow definition |

The raw `Function<Map, Map>` overload exists for backwards compatibility and trivial test stubs. In production case definitions it is wrong because:

1. **No execution pipeline.** `FuncWorkflowBuilder` wraps tasks in the quarkus-flow execution pipeline. This pipeline provides structured input/output mapping (`inputFrom` / `outputAs`), tracing hooks, and error handling. A raw lambda bypasses all of it.

2. **No composition.** A workflow can chain multiple tasks — fetch data, then process it, then call an agent. A raw lambda is a monolith. When the worker needs to do two things, a raw lambda forces you to put both in one block. A workflow lets you compose them as separate, named, independently traceable tasks.

3. **No backend portability.** The workflow execution model is designed to be backend-agnostic. Today tasks run in-process on Quartz threads. Tomorrow they may execute via Drools rules, OpenClaw skill invocations, or remote HTTP calls. A raw lambda is permanently in-process. A `FuncDSL.function()` task can be swapped for a `FuncDSL.agent()` or `FuncDSL.get()` without changing the worker structure.

4. **No observability.** Workflows carry task names that appear in traces and event logs. A raw lambda is anonymous — when something fails, the stack trace shows a synthetic lambda name, not a meaningful task identifier.

---

## FuncDSL Task Types

| Worker needs to... | Use | Example |
|--------------------|----|---------|
| Execute pure Java logic (computation, mapping, CDI service call) | `FuncDSL.function(lambda, InputType.class)` | Budget calculation, ledger write, commitment creation |
| Call an HTTP endpoint | `FuncDSL.get(name, url)` / HTTP equivalents | Fetch external data, check availability |
| Invoke an LLM/AI agent | `FuncDSL.agent(agentRef::method, RequestType.class)` | Sentiment analysis, document summarisation |
| Compose multiple steps | Chain tasks in `workflow().tasks(task1, task2, ...).build()` | Fetch data then process it |

### Input/Output Mapping

FuncDSL tasks support structured I/O mapping:

```java
function(logic, Map.class)
    .inputFrom("{ documentId: .documentId, content: .data.content }")
    .outputAs("{ summary: ., step: \"summarized\" }")
```

Use `inputFrom` to select specific context fields. Use `outputAs` to shape the output before it merges into the case context. These are JQ expressions — the same language used in YAML binding `inputMapping` / `outputMapping`.

---

## When Raw Lambdas Are Acceptable

**Unit tests only.** When constructing a `CaseDefinition` via the fluent DSL in a test class, a raw lambda is acceptable if:
- The test is verifying binding conditions, goal evaluation, or case flow — not worker execution
- The worker body is trivial (return a fixed map)
- The test does not exercise the workflow execution pipeline

Even in tests, prefer `workflow().tasks(function(...)).build()` when testing worker execution behaviour — the pipeline itself may have effects you want to verify.

---

## Relationship to case-definition-layers Protocol

Both protocols express the same principle at different levels:

| Level | Protocol | YAML path | Java path |
|-------|----------|-----------|-----------|
| Case definitions | PP-20260518 case-definition-layers | YAML resource + `YamlCaseHub` | Fluent DSL builders |
| Worker workflows | PP-20260531 worker-func-exec (this) | YAML workflow resource | Java FuncDSL |

The pairing rule applies at both levels. Workers are always added programmatically to `YamlCaseHub` subclasses (case definition YAML can't express worker implementations), but the worker's own execution logic uses quarkus-flow workflows — either YAML or FuncDSL.

---

## Violation Hints

- `Worker.builder().function((Map<String, Object> input) -> ...)` in production code (raw lambda, no workflow wrapper)
- A worker function that does multiple things (fetch + process + write) in a single lambda instead of chaining FuncDSL tasks
- Import of `Worker` without corresponding import of `FuncWorkflowBuilder` and `FuncDSL` in a production `CaseHub` class
- A `YamlCaseHub.augment()` method that adds workers with raw lambdas
- A worker workflow using FuncDSL for pure HTTP/data tasks with no YAML companion (where one could exist)
