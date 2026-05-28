---
id: PP-20260529-spi-adapter-placement
title: "SPI adapters start in the host repo as modules — extract to standalone repo only on confirmed cross-project adoption"
type: rule
scope: universal
applies_to: "Any multi-module Java project defining SPIs with pluggable adapter backends"
severity: important
refs:
  - docs/protocols/universal/module-tier-structure.md
created: 2026-05-29
---

# Protocol: SPI Adapter Module Placement — Evaluate In-Host-Repo First

**Applies to:** Any multi-module Java project with SPIs and pluggable adapters
**Severity:** Important — premature repo extraction adds ongoing overhead with no concrete benefit

---

## The Rule

New SPI adapters **start as submodules in the SPI's host repo**. They are extracted to a standalone
repo only when a confirmed consumer outside the host project exists.

**Start here:**
```
my-framework/
  api/                   ← SPI interface (pure Java, Tier 1)
  core/                  ← @DefaultBean no-op or default impl
  adapter-postgres/      ← Adapter starts as a submodule
  adapter-redis/         ← Adapter starts as a submodule
```

**Extract only when triggered:**
```
my-framework-adapters/   ← Created only when a cross-project consumer is confirmed
  adapter-postgres/
  adapter-redis/
```

---

## Rationale

The instinct to start adapters in a standalone repo comes from a valid concern: extraction cost.
Extracting a module later is mechanical but touches every consumer's `pom.xml`. However, this cost
only materialises after adoption has occurred. Before any adapters exist and before any external
consumers appear, repo creation is pure overhead: CI to wire, parent POM to set up, publish step
to manage.

The in-module pattern provides the same dependency isolation: consumers declare only the adapter
modules they need. No standalone repo is required to achieve this.

---

## Extraction Triggers

Extract to a standalone repo when either condition is met:

1. **Cross-project adoption** — a project outside the SPI's home org or domain needs the adapters without depending on the full host framework
2. **Operational divergence** — adapter complexity (release cadence, versioning, contributors) outgrows the host repo's scope

One condition is sufficient; both are not required.

---

## What Doesn't Change

- The SPI interface stays in the Tier 1 `api/` module regardless of adapter placement — see [module-tier-structure.md](module-tier-structure.md)
- The default no-op or `@DefaultBean` stays in the core module regardless
- When extraction does happen, the migration is mechanical: new repo, same module structure, consumer coordinate change

---

## Why This Is Easy to Get Wrong

The error pattern: a new SPI is designed, adapters are anticipated, a standalone adapter repo is created
*at design time* — before any adapter code exists and before any cross-project consumer is identified.
The repo exists as infrastructure with no content, adding ongoing CI and coordination overhead for no
current benefit.

The correct heuristic: **adapters without cross-project consumers justify a module, not a repo.**
A module costs nothing extra. A repo costs CI, publication, and coordination overhead from day one.
