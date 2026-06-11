---
id: PP-20260611-cc63b4
title: "JpaLedgerEntryRepository subclasses must qualify all EntityManager fields with @LedgerPersistenceUnit"
type: rule
scope: repo
applies_to: "Any class in casehub-engine-ledger that extends JpaLedgerEntryRepository and adds its own @Inject EntityManager field"
severity: critical
refs:
  - docs/protocols/casehub/ledger-subclass-extension.md
violation_hint: "Subclass injects @Inject EntityManager without @LedgerPersistenceUnit — findByCaseId or similar methods return empty results or wrong-datasource data in multi-datasource deployments (e.g. devtown with qhorus); single-datasource deployments show no symptoms"
created: 2026-06-11
---

`JpaLedgerEntryRepository` qualifies its `EntityManager em` field with `@LedgerPersistenceUnit` to target the named ledger datasource. CDI qualifier injection is not inherited — any subclass that adds its own `EntityManager` field for additional queries must replicate the `@LedgerPersistenceUnit` qualifier explicitly. Without it, the unqualified `@Inject EntityManager` resolves to the default persistence unit in multi-datasource deployments, silently routing queries to the wrong datasource. `CaseLedgerEntryRepository.caseEm` is the reference implementation and the prior violation.
