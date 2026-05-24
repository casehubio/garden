---
id: PP-20260524-075e06
title: "JDBC-only casehub harnesses must explicitly suppress reactive datasource in production application.properties"
type: rule
scope: application
applies_to: "all CaseHub harnesses running JDBC-only (no reactive URL configured) in production — clinical, aml, devtown"
severity: important
refs:
  - casehub-clinical: runtime/src/main/resources/application.properties
  - GE-20260524-cf232c (garden: quarkus-reactive-cdi-production)
violation_hint: "quarkus:build succeeds in @QuarkusTest but fails in production augmentation with 30+ unsatisfied CDI dependencies for ReactiveChannelService, ReactiveMessageStore, ReactiveInstanceService"
created: 2026-05-24
---

Transitive dependencies (casehub-qhorus, casehub-ledger) ship reactive-capable CDI beans that Quarkus indexes during production augmentation. When no reactive datasource URL is configured, the reactive implementations are `@Vetoed` — but the service beans that inject them are not guarded, leaving their injection points unsatisfied. The `@QuarkusTest` environment suppresses this via `quarkus.datasource.reactive=false` in test `application.properties`; without an explicit matching suppression in production `application.properties`, the production `quarkus:build` fails. Every JDBC-only harness must add `quarkus.datasource.reactive=false` and `quarkus.datasource.<named-datasource>.reactive=false` (one entry per named datasource) to `src/main/resources/application.properties`. Do not use `casehub.qhorus.reactive.enabled=false` — this key no longer exists in the qhorus config model.
