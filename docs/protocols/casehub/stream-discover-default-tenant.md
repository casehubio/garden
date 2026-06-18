---
id: PP-20260618-a7ad0d
title: "Stream modules must use DEFAULT_TENANT_ID in EndpointRegistry.discover() calls"
type: rule
scope: platform
applies_to: "All platform-streams-* modules that call EndpointRegistry.discover() to find their endpoints"
severity: critical
refs:
  - docs/superpowers/specs/2026-06-14-cloudEvent-streams-design.md
violation_hint: "Module passes PLATFORM_TENANT_ID or null as tenancyId — discover() returns zero results in standard single-tenant deployments; stream processors silently receive no endpoints at startup"
created: 2026-06-18
---

Stream modules (kafka, amqp, poll, camel) discover their endpoint descriptors via `EndpointRegistry.discover()`. The `tenancyId` argument must be `TenancyConstants.DEFAULT_TENANT_ID`. `matchesTenancy(d, DEFAULT_TENANT_ID)` returns descriptors registered under both `DEFAULT_TENANT_ID` (desiredstate-provisioned endpoints in a standard single-tenant deployment) and `PLATFORM_TENANT_ID` (platform-global endpoints) — so it correctly covers both cases. `PLATFORM_TENANT_ID = "platform"` (a string, not a UUID) passed as the tenancyId simplifies the filter to `d.tenancyId().equals("platform")` only — it returns zero results for the endpoints that desiredstate actually registers.
