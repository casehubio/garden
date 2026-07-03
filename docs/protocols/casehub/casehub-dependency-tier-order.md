---
id: PP-20260703-dependency-tier-order
title: "CaseHub dependency tier order — platform-api is the universal shared dependency"
type: rule
scope: platform
applies_to: "Any decision about where to place a shared type, SPI, or abstraction; any cross-repo dependency analysis"
severity: critical
refs:
  - docs/protocols/casehub/platform-api-scope.md
  - docs/protocols/universal/module-tier-structure.md
created: 2026-07-03
---

# Protocol: CaseHub Dependency Tier Order

**Applies to:** Any decision about where to place a shared type, SPI, or abstraction  
**Severity:** Critical — wrong placement creates circular dependencies or forces unnecessary coupling between peer repos

---

## The Dependency Graph

```
casehub-parent                    (BOM — version management only)
  └── casehub-platform-api        (zero casehub deps — the universal shared dependency)
        ├── casehub-worker-api
        ├── casehub-ledger-api
        ├── casehub-eidos-api
        ├── casehub-work-api
        ├── casehub-qhorus-api
        ├── casehub-iot-api
        ├── casehub-neocortex (rag-api)
        └── casehub-engine-api    (also depends on eidos-api, worker-api)
              └── casehub-blocks  (depends on engine-api, work-api, qhorus-api, eidos-api)
                    └── domain repos (aml, devtown, clinical, life, soc, iot, ...)
```

Every `-api` module in the graph depends on `casehub-platform-api`. Every runtime module
transitively depends on it. There is no repo in the CaseHub ecosystem that does not already
have `casehub-platform-api` on its classpath.

---

## The Key Implication

Adding a type to `casehub-platform-api` creates **no new dependency** for any repo. The
dependency already exists everywhere. Treating `casehub-platform-api` as a heavyweight or
risky dependency is wrong — it is the lightest module in the ecosystem: ~60 classes, zero
transitive dependencies, pure Java.

The `-api` modules at the next tier (worker-api, ledger-api, eidos-api, work-api, qhorus-api,
engine-api) are also lightweight pure-Java modules, but they DO create new cross-repo
dependencies when consumed. The decision "should repo X depend on Y-api?" is real for these
modules. It is not real for `casehub-platform-api`.

---

## Placement Decision Tree

When multiple repos need a shared type, SPI, or abstraction:

1. **Does a single domain own this concept?** → Put it in that domain's `-api` module.
   Other repos depend on that `-api`. This is the normal case.

2. **Do peer repos need it, but depending on each other would be wrong?** → Put it in
   `casehub-platform-api`. This is why platform-api exists: to break dependency cycles
   between peer repos that need the same concept.

3. **Is it a cross-cutting convention (marker interface, resolution framework) that multiple
   `-api` modules would implement?** → Put it in `casehub-platform-api`. The convention
   must be defined below all its consumers.

See `platform-api-scope.md` for the detailed scope criteria and examples.

---

## Peer Relationships (No Dependencies Between These)

These modules are peers — none may depend on another:

- `casehub-ledger-api`, `casehub-work-api`, `casehub-qhorus-api`, `casehub-eidos-api`,
  `casehub-worker-api`, `casehub-iot-api`

When two peer modules need a shared concept, it goes in `casehub-platform-api` — the only
module below all of them.

`casehub-engine-api` is NOT a peer of these — it sits above them and may depend on any of
them. But the peer `-api` modules must never depend on `casehub-engine-api`.

---

## Common Dependency Scenarios

| Scenario | Correct placement |
|----------|-------------------|
| Type needed by engine + work (peers for this purpose) | `casehub-platform-api` |
| SPI needed only by engine consumers | `casehub-engine-api` |
| Type needed only by eidos + engine | `casehub-eidos-api` (engine already depends on it) |
| Cross-cutting convention all `-api` modules implement | `casehub-platform-api` |
| Domain type specific to one vertical | That domain's `-api` module |
