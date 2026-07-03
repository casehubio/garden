---
id: PP-20260522-platform-api-scope
title: "casehub-platform-api — the universal shared dependency for cross-repo concepts"
type: rule
scope: platform
applies_to: "Any decision about where to place a new type, SPI, or record"
severity: important
refs:
  - docs/protocols/casehub/casehub-dependency-tier-order.md
  - docs/protocols/universal/module-tier-structure.md
  - docs/protocols/universal/maven-coordinate-standard.md
created: 2026-05-22
updated: 2026-07-03
---

# Protocol: casehub-platform-api Scope

**Applies to:** Any decision about where to place a new type, SPI, or record  
**Severity:** Important

---

## What casehub-platform-api Is

`casehub-platform-api` is the universal shared dependency in CaseHub. Every repo in the
ecosystem already depends on it. It is the lightest module (~60 classes, zero transitive
dependencies, pure Java) and sits at the bottom of the dependency graph — below all
domain `-api` modules.

It exists so that peer repos can share concepts **without depending on each other**.
Without it, each repo that needs `ActorType` or `Path` or `PreferenceKey` would define
its own incompatible version, or repos would need circular dependencies to share one.
`casehub-platform-api` breaks that problem.

Adding a type to `casehub-platform-api` creates no new dependency for any repo — the
dependency is already universal. See `casehub-dependency-tier-order.md` for the full
dependency graph.

---

## What Belongs Here

A type belongs in `casehub-platform-api` when:

1. **Multiple peer repos need it**, AND
2. **Those repos should not depend on each other** (because they're peers in the dependency
   graph, or because the dependency would be circular), AND
3. **Without a shared home, each repo would define its own incompatible version**

This includes:
- **Identity primitives** that cross all repos (`ActorType`, `CurrentPrincipal`)
- **Configuration mechanisms** used everywhere (`PreferenceKey`, `Preferences`)
- **Structural types** with no domain owner (`Path`, hierarchical scoping)
- **Cross-cutting SPIs** needed by multiple peer repos (`ActorStateContributor` — needed by
  ledger, work, qhorus, and engine; uses only stdlib types)
- **Shared conventions** that multiple `-api` modules implement (marker interfaces, resolution
  frameworks) — the convention must be defined below all its consumers

| Type | Belongs? | Reason |
|------|----------|--------|
| `ActorType` (HUMAN / AGENT / SYSTEM) | Yes | ledger, work, qhorus, engine all need it; these are peers |
| `CurrentPrincipal` | Yes | Identity crosses all repos; no single domain owns it |
| `Path` | Yes | Hierarchical scoping crosses all repos; no domain owner |
| `PreferenceKey` / `Preferences` | Yes | Configuration mechanism crosses all repos |
| `ActorStateContributor` | Yes | Needed by 4+ peer repos; uses only stdlib types |
| `NamedStrategy` / `StrategyResolver` | Yes | Cross-cutting convention that engine-api and work-api both implement |

---

## What Does NOT Belong Here

Types that have a clear domain owner go in that domain's `-api` module, even if multiple
repos consume them. The consuming repos add a dependency on the domain `-api` module —
these are lightweight pure-Java modules designed for exactly this purpose.

| Type | Belongs? | Reason |
|------|----------|--------|
| `AgentDescriptor` | No | Repos that need it depend on `casehub-eidos-api` |
| `WorkItem` | No | Repos that need it depend on `casehub-work-api` |
| `LedgerEntry` | No | Repos that need it depend on `casehub-ledger-api` |
| `CapabilityHealth` | No | Repos that need it depend on `casehub-eidos-api` |

The three-tier module structure (`api/` → `core/` → `runtime/`) already solves the
"lightweight dependency" problem. Every domain repo has a pure-Java `api/` module that
other repos can depend on without pulling in Quarkus, JPA, or runtime classes:

```
casehub-engine depends on casehub-eidos-api      ← lightweight SPI types only
devtown        depends on casehub-eidos (runtime) ← full registry + JPA
```

---

## The Anti-Pattern: Platform as Bucket

When domain types leak into `casehub-platform-api`, every repo gets unexpected transitive
exposure to unrelated concepts:

```
casehub-platform-api/
  ├── ActorType.java          ← ✅ cross-cutting primitive
  ├── CurrentPrincipal.java   ← ✅ cross-cutting primitive
  ├── AgentDescriptor.java    ← ❌ eidos domain type
  ├── WorkItemSummary.java    ← ❌ work domain type
  └── TrustQuery.java         ← ❌ ledger domain type
```

A change to an eidos type forces a publish of `casehub-platform-api`, which triggers
rebuilds across every repo. Domain types in platform-api create coupling that the
three-tier module structure is specifically designed to avoid.

---

## When a New Type Is Truly Platform-Primitive

If a new type genuinely belongs:

1. It has no meaningful existence outside of cross-cutting platform concerns
2. Placing it in any domain `-api` module would be awkward because it has no domain owner
3. It is needed by multiple repos that are peers in the dependency graph

Even then: consider whether a new, narrow submodule under `casehub-platform` is more
appropriate than adding to `casehub-platform-api` itself. Keep the module focused.
