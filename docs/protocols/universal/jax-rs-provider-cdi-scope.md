---
id: PP-20260530-2fd788
title: "JAX-RS @Provider beans in Quarkus must carry an explicit CDI scope annotation"
type: rule
scope: universal
applies_to: "Any Quarkus project using JAX-RS @Provider beans (ExceptionMapper, MessageBodyReader, ContainerRequestFilter, etc.)"
severity: guidance
violation_hint: "@Provider without @ApplicationScoped or @Singleton — bean is @Dependent-scoped; not wrong for stateless providers but non-canonical and inconsistent with every other CDI bean in the project"
created: 2026-05-30
---

All JAX-RS `@Provider` beans in Quarkus must carry an explicit CDI scope annotation alongside `@Provider`. For stateless mappers and filters, use `@ApplicationScoped`. `@Provider` alone gives `@Dependent` scope — Quarkus will instantiate the bean per injection point. This is functionally equivalent for stateless providers but is non-canonical and inconsistent with how all other CDI beans in the project are scoped; it surprises readers who expect an explicit scope on any managed bean.
