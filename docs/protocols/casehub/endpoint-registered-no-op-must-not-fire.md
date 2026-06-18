---
id: PP-20260618-f2d154
title: "EndpointRegistered must not fire from no-op EndpointRegistry implementations"
type: rule
scope: platform
applies_to: "Any class implementing EndpointRegistry — specifically NoOpEndpointRegistry @DefaultBean and any future no-op or test-only implementation"
severity: important
refs:
  - docs/superpowers/specs/2026-06-14-cloudEvent-streams-design.md
violation_hint: "NoOpEndpointRegistry.register() or a stub implementation calls fireAsync(EndpointRegistered) — platform-streams-camel and other EndpointRegistered observers build routes for phantom endpoints that are never actually stored"
created: 2026-06-18
---

`EndpointRegistered` is a CDI event that signals an endpoint was successfully stored. Only `EndpointRegistry` implementations that actually persist the descriptor to backing storage (in-memory, JPA, etc.) may fire it via `Event<EndpointRegistered>.fireAsync()`. The `NoOpEndpointRegistry @DefaultBean` is a silent no-op — its `register()` method must remain an empty no-op with no event firing. Firing from a no-op registry would trigger Camel route creation for phantom endpoints that have no real backing, causing spurious route failures and confusing logs.
