---
id: PP-20260603-a5e883
title: "All JPA repositories in casehub-persistence-hibernate must extend TenantAwareRepository"
type: rule
scope: repo
applies_to: "casehub-persistence-hibernate — all classes implementing engine SPIs via reactive JPA"
severity: critical
refs:
  - docs/protocols/casehub/tenancy-repository-pattern.md
  - docs/adr/0006-application-managed-rls-drop-and-create.md
violation_hint: "A class that extends AbstractJpaRepository directly (not TenantAwareRepository), or calls Panache.withSession() or Panache.withTransaction() directly inside a repository method, without the SET LOCAL injection"
created: 2026-06-03
---

Every JPA repository in `casehub-persistence-hibernate` must extend `TenantAwareRepository` and route all DB operations through `withTenantTransaction(work)` or `withCrossTenantTransaction(work)`. `withTenantTransaction()` wraps the work in a `Panache.withTransaction()` that sets `SET LOCAL "casehub.tenancy_id"` before any SQL — required for PostgreSQL Row Level Security. Using `Panache.withSession()` (autocommit mode) instead of `withTransaction()` means `SET LOCAL` has no effect: the variable resets before the next statement executes. Cross-tenant repositories (implementing `CrossTenantEventLogRepository`, `CrossTenantCaseInstanceRepository`) use `withCrossTenantTransaction()` which sets `SET LOCAL ROLE casehub_crosstenancy` instead.
