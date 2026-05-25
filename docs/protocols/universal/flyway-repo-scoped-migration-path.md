---
id: PP-20260525-607b33
title: "Flyway migrations must ship under a repo-scoped path — never the generic db/migration"
type: rule
scope: universal
applies_to: "All Quarkus modules shipping Flyway migrations as classpath resources"
severity: critical
refs:
  - docs/protocols/universal/flyway-migration-rules.md
violation_hint: "Module ships SQL files at src/main/resources/db/migration/ — the generic shared
  path; any other jar on the classpath with migrations at the same path causes Flyway startup
  failure with 'Found more than one migration with version N'"
created: 2026-05-25
---

Every module that ships Flyway migrations must place them under a path scoped to its own
name: `db/<reponame>/migration/` (e.g. `db/work/migration/`, `db/qhorus/migration/`,
`db/ledger/migration/`). The generic `classpath:db/migration` path is shared across every
jar on the classpath; when two modules both ship files there, Flyway sees duplicate version
numbers and fails at startup. Consumers reference each module by its scoped path explicitly
in `quarkus.flyway.locations`. Application repos that own domain migrations on the default
datasource place them under `db/<appname>/migration/` (e.g. `db/aml/migration/`) — never
at the generic `db/migration` root.
