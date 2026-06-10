---
id: PP-20260610-a4ecff
title: "JQ expressions must use panel-qualified paths after CaseContext panel migration"
type: rule
scope: platform
applies_to: "All CaseDefinition YAML bindings, goal conditions, milestone criteria, inputSchema, inputMapping, outputMapping; engine test JQ strings"
severity: critical
refs:
  - engine#80
violation_hint: "A JQ expression starting with `.` but not `.working.`, `.semantic.`, or `.episodic.` — e.g. `.result == null` instead of `.working.result == null`"
created: 2026-06-10
---

After engine#80, `CaseContext.asJsonNode()` returns the full panel document `{"working":{...},"semantic":{...},"episodic":{...}}` rather than a flat map. All JQ string expressions evaluated against this document must use panel-qualified paths. The working panel is accessed via `.working.key`; the semantic panel via `.semantic.key`; the episodic panel via `.episodic.workers`, `.episodic.milestones`, etc. Lambda `Predicate<CaseContext>` expressions are not affected — `context.get("key")` still delegates to the working panel unchanged. `CaseDefinitionYamlMapper` emits a parse-time WARNING (not an error) for expressions that start with `.` but are not panel-qualified, enabling staged migration without blocking load.
