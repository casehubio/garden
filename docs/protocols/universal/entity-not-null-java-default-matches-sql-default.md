---
id: PP-20260609-bacab1
title: "NOT NULL entity fields with a SQL DEFAULT must carry a matching Java-level field default"
type: rule
scope: universal
applies_to: "Any Hibernate/Panache @Entity class gaining a new NOT NULL column via a Flyway ALTER TABLE ADD COLUMN ... DEFAULT '...' migration — applies to both Quarkus drop-and-create test environments and production schema validation"
severity: important
refs:
  - docs/protocols/casehub/flyway-consumer-numbering.md
violation_hint: "New field has @Column(nullable = false) but no Java initialiser — existing test code that persists the entity without setting the field throws a NOT NULL constraint violation under drop-and-create, even though the SQL migration has a DEFAULT"
created: 2026-06-09
---

When a Flyway migration adds a `NOT NULL` column with a SQL `DEFAULT` value to an existing table, the corresponding Java entity field must also declare a matching Java-level default initialiser (e.g. `public String tenantId = "default";`). Hibernate's `drop-and-create` schema generation creates the column from entity metadata — it does NOT include the SQL `DEFAULT` clause from the migration, because drop-and-create bypasses Flyway entirely. Any test that persists the entity without explicitly setting the new field will hit a NOT NULL constraint violation. The Java-level default closes this gap: it satisfies the database constraint automatically, making all existing persist sites backward-compatible without requiring each to be updated. The SQL `DEFAULT` remains necessary for safe production migration of existing rows; the Java default is the equivalent safety valve for the test environment.
