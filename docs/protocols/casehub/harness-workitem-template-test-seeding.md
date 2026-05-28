---
id: PP-20260528-913df2
title: "Seed WorkItemTemplates in @QuarkusTest via @BeforeEach @Transactional — Flyway V-seeds don't run in test mode"
type: rule
scope: application
applies_to: "Any casehub harness @QuarkusTest that uses WorkItemTemplate-based task creation endpoints"
severity: important
refs:
  - docs/protocols/casehub/HARNESS-INDEX.md
  - docs/protocols/universal/quarkus-test-database.md
violation_hint: "POST /life-tasks (or equivalent) returns 422 'Unknown templateRef' in tests even though a Flyway data migration seeds the template in production — Flyway is disabled in test config (drop-and-create mode)"
created: 2026-05-28
---

Casehub harness tests use H2 drop-and-create (Flyway disabled) to avoid V-number collisions between foundation modules. Flyway data migrations that seed `WorkItemTemplate` rows (e.g. `V102__life_workitem_template_seeds.sql`) never run in tests. Any `@QuarkusTest` that exercises an endpoint backed by `WorkItemTemplateService.findByRef()` will get a 422 unless the template is seeded in the test setup. The pattern: a non-static `@BeforeEach @Transactional void seedTemplates()` instance method that calls `WorkItemTemplate.persist()` guarded by `if (WorkItemTemplate.find("name", name).count() == 0)`. The guard prevents duplicate insertion when multiple test methods share the same Quarkus application context (the application is started once; `@BeforeEach` runs before every test method). Static `@BeforeAll` methods with `@Transactional` are not intercepted by CDI — use `@BeforeEach` instance methods instead (PP-20260528-3d3847).
