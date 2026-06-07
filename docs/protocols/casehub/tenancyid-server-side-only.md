---
id: PP-20260607-69eba2
title: "tenancyId is server-side infrastructure — never expose in client-facing APIs"
type: rule
scope: platform
applies_to: "all casehubio modules — REST request/response DTOs, JSON schemas, OpenAPI specs, MCP tool parameters, SSE client endpoints"
severity: critical
refs:
  - docs/protocols/casehub/tenancy-repository-pattern.md
  - docs/protocols/casehub/no-conditional-tenancy-filtering.md
violation_hint: "A REST response DTO, request DTO, JSON Schema, or OpenAPI spec that includes a tenancyId field — or a client-side SDK type that accepts or returns tenancyId"
created: 2026-06-07
---

tenancyId is established by the authenticated session on the server side — clients never send it, never see it, and never need to know it exists. It must not appear in REST request DTOs, REST response DTOs, JSON Schemas exposed to clients, OpenAPI specifications, MCP tool input/output schemas, or any client-facing SDK type. It belongs exclusively on JPA entities, internal CDI events, and server-to-server wire formats (e.g. distributed SSE relay between cluster nodes). The entity-to-response mapper is the boundary: it maps tenancyId in, never out. Violation creates a client-observable surface that leaks tenant isolation semantics and invites callers to attempt cross-tenant operations by manipulating the field.
