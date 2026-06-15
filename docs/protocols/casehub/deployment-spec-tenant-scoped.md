---
id: PP-20260615-c32b1b
title: "Deployment specifications are tenant-scoped — tenancyId is a required first-class field"
type: rule
scope: platform
applies_to: "Any deployment spec (CasehubDeploymentSpec or equivalent) submitted to ReconciliationLoop.start() — casehub-ops and any future deployment-layer component"
severity: important
refs:
  - docs/superpowers/specs/2026-06-13-p0-layering-decisions-design.md
violation_hint: "Deployment spec has no tenancyId field — bootstrap code must guess a tenant (e.g. TenancyConstants.DEFAULT_TENANT_ID) or iterate ambiguously 'for each tenant in the spec'; ReconciliationLoop.start(tenancyId, graph) has no tenant to pass"
created: 2026-06-15
---

`ReconciliationLoop` is per-tenant by design — `ReconciliationLoop.start(String tenancyId, DesiredStateGraph desired)` requires `tenancyId` as a first-class parameter. Any deployment configuration submitted to it is inherently scoped to exactly one tenant. Every deployment spec record or YAML artifact must carry an explicit `tenancyId` field. Phrasing like "for each tenant in the spec" is a symptom of a missing field: if the spec has no `tenancyId`, a bootstrap bean cannot implement multi-tenant deployment without inventing a tenant source that doesn't exist. Single-tenant deployments use `TenancyConstants.DEFAULT_TENANT_ID` explicitly in the YAML — they do not rely on an implicit default.
