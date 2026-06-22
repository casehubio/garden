---
id: PP-20260622-44f497
title: "RBAC classifier context guard — catch ContextNotActiveException at groups() call sites; pass pre-computed role flags to avoid redundant lookups"
type: rule
scope: application
applies_to: "@ApplicationScoped ActionRiskClassifier (or similar) beans that inject CurrentPrincipal for role-differentiated behaviour in casehub harness applications"
severity: important
refs:
  - casehub/auth-retrofit-readiness.md
violation_hint: "Using Arc.container().requestContext().isActive() instead of try/catch (couples to Quarkus internals, breaks in plain Mockito tests); calling principal.groups() multiple times per classification (redundant proxy invocations, potential inconsistency)"
created: 2026-06-22
---

`@ApplicationScoped` beans that inject a `@RequestScoped CurrentPrincipal` and are called from async/scheduler contexts (where no HTTP request context is active) must guard each `principal.groups()` call with `try { ... } catch (ContextNotActiveException e) { return false; }` at the call site — never via `Arc.container().requestContext().isActive()`, which couples to Quarkus internals and cannot be tested in plain Mockito unit tests (Arc.container() returns null without a CDI container). Where multiple role checks are needed in a single classification path (e.g. `isAdmin()` then `isJunior(admin)`), compute the admin flag once and pass it as a parameter to downstream helpers to avoid redundant `groups()` proxy invocations; `isJunior(boolean admin)` short-circuits immediately when `admin=true`, eliminating the second lookup. The context-inactive fallback must default to the most restrictive non-always-gate tier (typically member threshold) — not `always-gate`, which would block background workers from acting autonomously on routine operations.
