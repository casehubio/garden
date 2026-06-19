---
id: PP-20260619-eee0fd
title: "TTL on accumulator SPI entries belongs in the store implementation, not the SPI signature"
type: rule
scope: platform
applies_to: "SPI interfaces that accumulate time-bounded entries (e.g. CapabilitySpecializationStore, AgentStateStore)"
severity: important
refs:
  - ../../../docs/PLATFORM.md
violation_hint: "SPI method carries an Instant expiresAt parameter for a policy-driven expiry (not an event-provided one); forces callers to know routing-policy TTL values they have no business knowing"
created: 2026-06-19
---

When an SPI accumulates time-bounded entries (e.g. decline counts, degradation state), the TTL belongs in the store implementation via `@ConfigProperty` — not on the SPI signature. The caller records the event; the store decides how long to remember it. Exception: when the TTL is event-specific and the caller has the exact expiry from an external source (e.g. a rate-limit `Retry-After` header), the TTL goes on the SPI signature because the caller — not the store — is the authoritative source of the expiry. The distinction: `AgentStateStore.record(agentId, tenancyId, reason, Instant expiresAt)` carries `expiresAt` because the engine receives the exact retry window from the rate-limit event; `CapabilitySpecializationStore.recordDecline(agentId, tenancyId, capabilityName, domain)` has no `expiresAt` because the TTL is a routing policy owned by eidos, not a fact supplied by the casehub-ledger caller.
