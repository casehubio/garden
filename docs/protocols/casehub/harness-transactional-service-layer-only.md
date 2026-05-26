---
id: PP-20260526-75d9c9
title: "@Transactional belongs on service methods only — never on REST resource methods"
type: rule
scope: application
applies_to: "All casehub harness applications with a service + REST resource layer"
severity: important
refs:
  - docs/protocols/casehub/hexagonal-application-service-placement.md
violation_hint: "Resource method and service method both carry @Transactional — the service's REQUIRED transaction joins the resource's transaction, hiding TOCTOU races on check-then-act operations (e.g. existence check followed by delete in separate transaction phases)."
created: 2026-05-26
---

In casehub harness applications, `@Transactional` belongs on service methods that write to the database — not on REST resource methods. Resource methods delegate to services; they do not own transaction boundaries. When a resource method carries `@Transactional`, the CDI container may split the logical operation across multiple transactions (one for read, one for write in separate service calls), enabling TOCTOU races. The canonical example: a resource that checks entity existence with an un-annotated `findById()`, then calls a `@Transactional` `delete()` — the existence check runs outside any transaction, and a concurrent deletion between the two calls produces a phantom 204 response. Consolidating the existence check inside the service's `@Transactional delete()` boundary eliminates the race. Resource methods remain free of `@Transactional`; they let JAX-RS exception mapping handle 404 and 409 responses thrown by the service.
