---
id: PP-20260531-worker-func-exec
title: "Worker functions must use FuncWorkflowBuilder — never raw lambdas in production case definitions"
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

## The Rule

Worker functions in production case definitions MUST use `FuncWorkflowBuilder.workflow().tasks(...).build()` to compose `FuncDSL` tasks. Do not pass a raw `Function<Map<String, Object>, Map<String, Object>>` lambda to `Worker.Builder.function()`.

### Correct

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

### Incorrect

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

## Why This Matters

`Worker.Builder.function()` accepts four types:

| Type | Import | What it is |
|------|--------|------------|
| `Workflow` | `FuncWorkflowBuilder.workflow()` | Composed FuncDSL task pipeline — the standard execution model |
| `Function<Map, Map>` | raw lambda | Bare function — no pipeline, no tracing, no composition |
| `Agent` | `FuncDSL.agent()` | LLM/AI agent task — structured agent invocation |
| `File` | file reference | File-based workflow definition |

The raw `Function<Map, Map>` overload exists for backwards compatibility and trivial test stubs. In production case definitions it is wrong because:

1. **No execution pipeline.** `FuncWorkflowBuilder` wraps tasks in the quarkus-flow execution pipeline. This pipeline provides structured input/output mapping (`inputFrom` / `outputAs`), tracing hooks, and error handling. A raw lambda bypasses all of it.

2. **No composition.** A workflow can chain multiple tasks — fetch data, then process it, then call an agent. A raw lambda is a monolith. When the worker needs to do two things, a raw lambda forces you to put both in one block. A workflow lets you compose them as separate, named, independently traceable tasks.

3. **No backend portability.** The workflow execution model is designed to be backend-agnostic. Today tasks run in-process on Quartz threads. Tomorrow they may execute via Drools rules, OpenClaw skill invocations, or remote HTTP calls. A raw lambda is permanently in-process. A `FuncDSL.function()` task can be swapped for a `FuncDSL.agent()` or `FuncDSL.get()` without changing the worker structure.

4. **No observability.** Workflows carry task names that appear in traces and event logs. A raw lambda is anonymous — when something fails, the stack trace shows a synthetic lambda name, not a meaningful task identifier.

---

## Choosing the Right FuncDSL Task Type

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

This protocol governs HOW worker functions are implemented. The case-definition-layers protocol (PP-20260518) governs WHERE case definitions are authored (YAML vs fluent DSL). Both apply simultaneously:

- A YAML case definition + `YamlCaseHub` subclass augments with workers using `workflow().tasks(...)` — workers are always programmatic (lambdas aren't YAML-expressible)
- A fluent DSL case definition uses the same `workflow().tasks(...)` for worker functions
- Both production paths use the same execution model

---

## Violation Hints

- `Worker.builder().function((Map<String, Object> input) -> ...)` in production code (raw lambda, no workflow wrapper)
- A worker function that does multiple things (fetch + process + write) in a single lambda instead of chaining FuncDSL tasks
- Import of `Worker` without corresponding import of `FuncWorkflowBuilder` and `FuncDSL` in a production `CaseHub` class
- A `YamlCaseHub.augment()` method that adds workers with raw lambdas
