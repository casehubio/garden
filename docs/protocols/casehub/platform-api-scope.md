---
id: PP-20260522-platform-api-scope
title: "casehub-platform-api is for foundational primitives only — not a shared types bucket"
type: rule
scope: platform
applies_to: "Any decision about where to place a new type, SPI, or record"
severity: important
refs:
  - docs/protocols/universal/module-tier-structure.md
  - docs/protocols/universal/maven-coordinate-standard.md
created: 2026-05-22
---

# Protocol: casehub-platform-api Scope — Foundational Primitives Only

**Applies to:** Any decision about where to place a new type, SPI, or record  
**Severity:** Important — placing domain types in `casehub-platform-api` turns it into a
shared bucket, increases coupling, and defeats the purpose of domain-specific `*-api` modules

---

## The Purpose

`casehub-platform-api` exists to **avoid duplication of shared concepts across repos, without
forcing those repos to depend on each other.**

Without `casehub-platform-api`, each repo that needs `ActorType` would define its own version.
ledger's `ActorType`, work's `ActorType`, and qhorus's `ActorType` would be incompatible. The
repos would either duplicate the type or create circular dependencies to share it.
`casehub-platform-api` breaks that problem — all repos share one definition without depending on
each other.

This is the only reason to put something in `casehub-platform-api`.

---

## The Rule

A type belongs in `casehub-platform-api` if and only if:

1. **Multiple repos need it**, AND
2. **Those repos should not depend on each other** (because they're peers, or because the
   dependency would be circular or architecturally wrong), AND
3. **Without a shared home, each repo would define its own incompatible version**

If repos that need the type can simply depend on a single domain's `*-api` module — that is
the right answer. `casehub-platform-api` is not needed.

---

## Test: Does This Belong in `casehub-platform-api`?

| Type | Belongs in platform-api? | Reason |
|------|--------------------------|--------|
| `ActorType` (HUMAN / AGENT / SYSTEM) | ✅ Yes | ledger, work, qhorus, engine all need it; these repos should not depend on each other |
| `CurrentPrincipal` | ✅ Yes | Identity crosses all repos; no single domain owns it |
| `Path` | ✅ Yes | Hierarchical scoping crosses all repos; no domain owner |
| `PreferenceKey` / `Preferences` | ✅ Yes | Configuration mechanism crosses all repos |
| `AgentDescriptor` | ❌ No | Repos that need it (devtown, engine, claudony) can depend on `casehub-eidos-api` |
| `WorkItem` | ❌ No | Repos that need it depend on `casehub-work-api` |
| `LedgerEntry` | ❌ No | Repos that need it depend on `casehub-ledger-api` |
| `CapabilityHealth` | ❌ No | Repos that need it (engine) depend on `casehub-eidos-api` |

---

## The Lightweight Dependency Problem Is Already Solved

The common mistake: placing a type in `casehub-platform-api` because "other repos need it
but don't want to pull in the full runtime."

This problem is solved by the **three-tier module structure**. Every domain repo has a
pure-Java `api/` module (Tier 1 — no Quarkus, no JPA) that other repos can depend on
without pulling in the full runtime.

```
casehub-engine depends on casehub-eidos-api      ← lightweight SPI types only
devtown        depends on casehub-eidos (runtime) ← full registry + JPA
```

This is identical to how engine already handles other domains:

```
casehub-engine depends on casehub-work-api      ← WorkBroker SPI, no JPA
casehub-engine depends on casehub-ledger-api    ← audit types, no JPA
```

`casehub-platform-api` is not needed to solve this. The `*-api` module pattern already
provides the right granularity.

---

## Anti-Pattern: The Platform Bucket

```
casehub-platform-api/
  ├── ActorType.java          ← ✅ foundational primitive
  ├── CurrentPrincipal.java   ← ✅ foundational primitive
  ├── AgentDescriptor.java    ← ❌ Eidos domain type
  ├── CapabilityHealth.java   ← ❌ Eidos domain SPI
  ├── WorkItemSummary.java    ← ❌ work domain type
  └── TrustQuery.java         ← ❌ ledger domain type
```

When `casehub-platform-api` contains domain types, every repo that depends on it for
foundational primitives gets unexpected transitive dependencies on unrelated domains.
It also creates implicit coupling — a change to an Eidos type forces a publish of
`casehub-platform-api`, which triggers rebuilds across every repo in the platform.

---

## Correct Pattern

Domain types that multiple repos reference go in the domain's own `api/` module:

```
casehub-eidos/
  api/              ← casehub-eidos-api: AgentDescriptor, Vocabulary, CapabilityHealth, etc.
  runtime/          ← casehub-eidos: JPA registry, ClaudeMarkdownRenderer, etc.
  deployment/       ← casehub-eidos-deployment: @BuildStep processors

casehub-platform/
  platform-api/     ← casehub-platform-api: ActorType, CurrentPrincipal, Path, PreferenceKey ONLY
```

---

## When a New Type Is Truly Platform-Primitive

If a new type genuinely belongs in `casehub-platform-api`:

1. It has no meaningful existence outside of cross-cutting platform concerns
2. Placing it in any domain `api/` module would be awkward because it has no domain owner
3. It is needed by multiple repos that have no dependency relationship with each other

Even then: consider whether a new, narrow module under `casehub-platform` is more appropriate
than adding to `casehub-platform-api`. The platform package is already the exception; keep it
from growing.
