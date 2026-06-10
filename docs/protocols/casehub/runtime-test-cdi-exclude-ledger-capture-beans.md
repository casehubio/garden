---
id: PP-20260610-18a084
title: "Exclude casehub-engine-ledger CDI capture beans from runtime test application.properties when ledger Flyway migrations are absent"
type: rule
scope: repo
applies_to: "casehub-engine runtime test profile (and any module with casehub-engine-ledger as a test dependency but without ledger Flyway migrations)"
severity: critical
refs:
  - runtime/src/test/resources/application.properties
violation_hint: "Tests time out in unrelated cases (SimpleCaseHubBeanTest, SpiWiringIntegrationTest) with buried WARN logs: 'CaseLifecycleEvent observer failed' and 'ERROR: relation ledger_subject_sequence does not exist'"
garden_ref: "GE-20260610-1c73c1"
created: 2026-06-10
---

The `casehub-engine-ledger` module is a test dependency of `casehub-engine` (runtime). Its CDI beans (`CaseLedgerEventCapture`, `WorkerDecisionEventCapture`) are activated automatically and observe every `CaseLifecycleEvent`. These beans call `JpaLedgerEntryRepository` which, as of casehub-ledger SNAPSHOT post-20260529, issues MERGE statements against `ledger_subject_sequence` — a table created only by the ledger's Flyway migrations. Since runtime tests use `quarkus.flyway.migrate-at-start=false` and a drop-and-create engine schema, this table does not exist. The observer failure is silent from the test's perspective but prevents cases from completing, causing cascading timeouts. Fix: add both beans to `quarkus.arc.exclude-types` in `runtime/src/test/resources/application.properties`.
