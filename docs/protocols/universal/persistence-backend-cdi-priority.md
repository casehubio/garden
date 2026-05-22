---
id: PP-20260522-0cfa30
title: "Three-tier CDI priority ladder for persistence backends"
type: rule
scope: universal
applies_to: "Any Quarkus project shipping multiple competing CDI implementations of the same persistence SPI"
severity: important
refs:
  - docs/protocols/casehub/platform-spi-contract.md
  - docs/protocols/universal/module-tier-structure.md
created: 2026-05-22
---

# Protocol: Three-Tier CDI Priority Ladder for Persistence Backends

**Applies to:** Any Quarkus project with a persistence SPI that has more than one backend
implementation — JPA/SQL as primary, NoSQL as secondary, mock as default.

---

## The Ladder

| Tier | Annotation | CDI resolution | Purpose |
|------|-----------|---------------|---------|
| 1 — Mock / no-op | `@DefaultBean` | Yielded by any non-default bean | Ships with SPI; active when no real backend is on the classpath |
| 2 — Primary backend | `@ApplicationScoped` | Beats `@DefaultBean` | JPA / SQL implementation |
| 3 — Secondary backend | `@Alternative @Priority(1)` | Beats `@ApplicationScoped` | NoSQL implementation; wins when co-deployed with JPA |

CDI resolution order is deterministic: `@Alternative @Priority(1)` > `@ApplicationScoped` > `@DefaultBean`. No consumer-side configuration is required — activation is purely classpath-driven.

**Quarkus ARC dependency:** This ladder relies on Quarkus ARC's automatic global activation of `@Priority`-annotated `@Alternative` beans. Standard CDI 4.x (Jakarta EE) requires `beans.xml` or `@EnabledAlternatives` to activate alternatives. The co-deployment guarantee described here applies only to Quarkus / ARC deployments.

---

## Co-Deployment Guarantee

Adding a secondary backend module to the classpath silently promotes it to the active
implementation. Removing it silently falls back to the primary. No consumer code changes,
no `application.properties` toggles, no `@IfBuildProperty` guards on the consumer side.

This is the design intent: **additive classpath activation**. Each new backend is an
independent artifact. Consumers opt in by declaring a Maven dependency — nothing else.

Note: the secondary backend beats `@DefaultBean` whether or not the primary is present. JPA
is not a prerequisite for NoSQL activation — a MongoDB-only deployment with no JPA module
is valid and works correctly.

---

## Per-Tier Implementation Guide

### Tier 1 — Mock / no-op default

Ships in the same module as the SPI (or a dedicated `platform`/`runtime` module):

```java
@DefaultBean
@ApplicationScoped
public class MockPreferenceProvider implements PreferenceProvider {
    // Configurable via @ConfigProperty — returns sensible no-op defaults
}
```

Rules:
- Must be `@DefaultBean` — never bare `@ApplicationScoped` (would collide with real backends)
- Must be `@ApplicationScoped` — mock has no request context to read from
- Must be configurable via `@ConfigProperty` so tests can drive behaviour without a real backend

### Tier 2 — Primary backend (JPA / SQL)

Ships in a dedicated `persistence-jpa` or equivalent module:

```java
@ApplicationScoped   // no @DefaultBean — beats the mock automatically
public class JpaPreferenceProvider implements PreferenceProvider {
    @Inject EntityManager em;
    // ...
}
```

Rules:
- Plain `@ApplicationScoped`, no `@DefaultBean` — CDI picks this over the mock automatically
- Do **not** use `@Alternative` — this is the primary; it must be active without extra configuration
- Ship Flyway migrations inside the module; declare location in the module's own config or
  instruct consumers to add the classpath location explicitly

### Tier 3 — Secondary backend (NoSQL / alternative)

Ships in a dedicated `persistence-mongodb` or equivalent module:

```java
@Alternative
@Priority(1)
@ApplicationScoped
public class MongoPreferenceProvider implements PreferenceProvider {
    @Inject MongoClient mongo;
    // ...
}
```

Rules:
- Must be `@Alternative @Priority(1)` — this is what lets it beat the JPA implementation
- `@Priority(1)` is the standard value; do not use higher values unless a fourth tier is needed
  (consult the SPI owner before introducing a fourth tier)
- Do **not** use `@DefaultBean` — `@Alternative` already provides CDI selection control

---

## What NOT To Do

**Do not use `@Alternative @Priority(1)` on a primary (JPA) backend.**
A JPA impl annotated `@Alternative @Priority(1)` still beats the mock — it is active. The
failure occurs when a NoSQL secondary module (also `@Priority(1)`) is added: ARC sees two
`@Alternative @Priority(1)` beans for the same type and throws an ambiguous dependency exception
at startup. Use plain `@ApplicationScoped` for the primary so the secondary can safely claim
`@Priority(1)` without collision.

**Do not use `@DefaultBean` on a real (non-mock) implementation.**
`@DefaultBean` yields to any other qualifying bean, including other real implementations.
If two real backends both use `@DefaultBean`, CDI resolution becomes ambiguous and deployment fails.

**Do not add `@Alternative` to the mock.**
The mock is already `@DefaultBean` — that is sufficient. Adding `@Alternative` changes
its activation semantics and breaks the ladder.

**Do not use `@IfBuildProperty` to switch backends on the consumer side.**
The ladder eliminates the need for build-time flags at call sites. Backend selection is
entirely classpath-driven. Build-time flags belong only inside a backend module that
ships both reactive and blocking service beans (see `reactive-service-build-gating.md`).

---

## Test Isolation

`@QuarkusTest` loads every `@Alternative` on the augmentation classpath. If both JPA and
NoSQL modules are on the test classpath, the NoSQL backend wins (Tier 3 > Tier 2).

To test a specific backend in isolation:
- Scope test dependencies correctly — only the module under test and its direct deps on the test classpath
- Do not add competing backend modules as `test` scope dependencies unless you intend to test co-deployment
- If classpath isolation is not possible (e.g. a shared integration-test module), use
  `quarkus.arc.selected-alternatives=<fully.qualified.ClassName>` in `application.properties`
  to force a specific backend and override ARC's priority-based resolution

The mock (`@DefaultBean`) is active automatically in any test module that has the SPI on the
classpath but no real backend. This is the correct default for pure-Java / unit tests.
