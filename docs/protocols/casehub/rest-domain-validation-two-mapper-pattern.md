---
id: PP-20260530-5354d0
title: "Domain validation errors from record compact constructors require two JAX-RS exception mappers in tandem"
type: rule
scope: application
applies_to: "CaseHub application REST layer using record compact constructors for domain validation"
severity: important
refs:
  - jvm/GE-20260530-3562b0.md
violation_hint: "Only ExceptionMapper<IllegalArgumentException> registered — POST with invalid JSON body returns 500 or an unformatted Jackson error instead of 400 with the domain message"
created: 2026-05-30
---

Register two `@Provider @ApplicationScoped` exception mappers: `ExceptionMapper<IllegalArgumentException>` for direct service-layer throws, and `ExceptionMapper<JsonMappingException>` for Jackson-wrapped compact constructor violations. When a record compact constructor throws `IllegalArgumentException` during deserialization, Jackson rethrows it as `ValueInstantiationException extends JsonMappingException` — a single `ExceptionMapper<IllegalArgumentException>` never fires for that path. The `JsonMappingException` mapper must inspect `getCause()`; if the cause is `IllegalArgumentException`, return 400 with the cause message; for all other causes, return a generic 400 without surfacing Jackson internals.
