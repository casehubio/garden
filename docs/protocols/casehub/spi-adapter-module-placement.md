---
id: PP-20260529-spi-adapter-placement
title: "SPI adapters start in-platform as modules — extract to standalone repo only on confirmed cross-ecosystem adoption"
type: rule
scope: platform
applies_to: "Any new SPI adapter module whose consumers at design time are all within CaseHub"
severity: important
refs:
  - docs/protocols/universal/module-tier-structure.md
  - docs/protocols/casehub/platform-api-scope.md
created: 2026-05-29
---

# Protocol: SPI Adapter Module Placement — Evaluate In-Platform First

**Applies to:** Any new SPI adapter whose consumers are CaseHub-internal at design time
**Severity:** Important — premature repo extraction adds ongoing overhead with no concrete benefit

---

## The Rule

New SPI adapters **start as submodules in the SPI's host repo**. They are extracted to a standalone
repo only when a confirmed non-CaseHub consumer exists.

**Start here:**
```
casehub-platform/
  platform-api/          ← SPI interface (pure Java)
  platform/              ← @DefaultBean no-op
  memory-memori/         ← Adapter starts as a submodule
  memory-mem0/           ← Adapter starts as a submodule
```

**Extract only when triggered:**
```
casehub-memory/          ← Created only after a non-CaseHub consumer is confirmed
  memory-memori/
  memory-mem0/
```

---

## Rationale

The instinct to start in a standalone repo comes from a valid concern: extraction cost. The
`casehub-eidos` precedent showed that extracting a module later is mechanical but touches every
consumer's `pom.xml`. However, this cost only materialises if adoption has already occurred.
Before any adapters exist and before any non-CaseHub consumers appear, repo creation is pure
overhead: CI to wire, parent POM to set up, publish step to manage, cross-repo issue tracking
to maintain.

The in-platform module pattern (`persistence-jpa/`, `persistence-mongodb/`) is already proven.
Module separation provides dependency isolation — consumers declare only the modules they need.
This is equivalent to what a standalone repo provides.

---

## Extraction Triggers

Extract to a standalone repo when either condition is met:

1. **Cross-ecosystem adoption** — a non-CaseHub project needs the adapters without pulling in `casehub-platform`
2. **Operational divergence** — adapter complexity (release cadence, versioning, contributors) outgrows the host repo's scope

One condition is sufficient; both are not required.

---

## What Doesn't Change

- The SPI interface stays in `platform-api` (Tier 1 — pure Java) regardless of adapter placement
- The `@DefaultBean` no-op stays in `platform` regardless
- Module tier structure rules still apply — adapter modules may use Quarkus and REST clients, but not in `platform-api`
- When extraction does happen, the migration is mechanical: new repo, same module structure, consumer coordinate change

---

## Origin

This rule emerged from ADR-0008 (CaseMemoryStore adapter placement, casehub-platform). The original
decision chose a standalone repo preemptively. With zero adapters built and no non-CaseHub consumers,
this was reversed in favour of in-platform modules. See ADR-0008 amendment (2026-05-29).
