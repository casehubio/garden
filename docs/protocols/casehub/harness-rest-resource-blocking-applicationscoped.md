---
id: PP-20260526-d0b921
title: "Harness REST resource classes must be @Blocking @ApplicationScoped when using quarkus-rest with JDBC ORM"
type: rule
scope: application
applies_to: "All casehub harness applications using quarkus-rest (RESTEasy Reactive) with JDBC Panache or Hibernate ORM"
severity: important
refs:
  - docs/protocols/universal/reactive-vs-blocking-selection.md
  - docs/protocols/casehub/auth-retrofit-readiness.md
violation_hint: "@QuarkusTest passes without @Blocking — the I/O thread is silently blocked; only visible as throughput degradation under production load. Missing @ApplicationScoped prevents CDI interceptor injection needed for auth retrofit."
created: 2026-05-26
---

Every REST resource class in a casehub harness application that uses `quarkus-rest` with JDBC ORM (Panache, Hibernate ORM) must carry `@Blocking @ApplicationScoped` at the class level. `@Blocking` is required because `quarkus-rest` (RESTEasy Reactive) runs on the Vert.x I/O thread by default — JDBC calls block the I/O thread and degrade event-loop throughput. `@ApplicationScoped` is required for auth-retrofit-readiness: CDI interceptors for `@RolesAllowed` and tenancy filters can only be woven onto beans with a proper CDI scope. `@QuarkusTest` does not enforce reactive thread discipline, so the absence of `@Blocking` produces no test failure — the violation is only observable under concurrent production load.
