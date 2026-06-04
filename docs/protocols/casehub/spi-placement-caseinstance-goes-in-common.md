---
id: PP-20260604-60bc5a
title: "SPIs using CaseInstance or common/internal types must go in common/spi/, not api/spi/"
type: rule
scope: platform
applies_to: "All SPI interfaces in casehub-engine modules"
severity: important
refs:
  - casehub-engine-common/src/main/java/io/casehub/engine/common/spi/WorkOrchestrator.java
  - docs/CLAUDE.md — SPI placement rule
violation_hint: "Placing an interface that takes CaseInstance in api/spi/ — causes circular dependency api ← common ← api at compile time"
created: 2026-06-04
---

Operational SPIs (worker provisioning, lifecycle, channels) normally go in `api/spi/`; persistence SPIs (`CaseMetaModelRepository`, etc.) go in `casehub-engine-common/spi/`. Exception: if an SPI declares a method that takes `CaseInstance` or any other type from `casehub-engine-common/internal/`, the interface must go in `common/spi/` regardless of its conceptual category. Placing it in `api/spi/` creates a circular dependency: `api` would need to import from `common`, but `common` already imports from `api` (`WorkResult`, `WorkRequest`, etc.). `WorkOrchestrator` is the reference example — it is an operational SPI but lives in `common/spi/` because both its methods take `CaseInstance`. Compile-time enforcement: building with `casehub-engine-api` will fail if it imports `casehub-engine-common` types.
