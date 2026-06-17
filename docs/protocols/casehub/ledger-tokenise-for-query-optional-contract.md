---
id: PP-20260617-4d345f
title: "ActorIdentityProvider.tokeniseForQuery() Optional semantics — empty=null input, present=always query"
type: rule
scope: platform
applies_to: "casehub-ledger and any consumer providing a custom ActorIdentityProvider implementation"
severity: important
refs:
  - api/src/main/java/io/casehub/ledger/api/spi/ActorIdentityProvider.java
violation_hint: "A custom ActorIdentityProvider implementation returns Optional.empty() to mean 'no token exists' — causing findByActorId and findAttestationsByAttestorId to silently return empty results for SYSTEM/AGENT actors whose entries ARE in the ledger under the raw actorId"
created: 2026-06-17
---

`ActorIdentityProvider.tokeniseForQuery()` returns `Optional<String>` with these exact semantics: `Optional.empty()` signals **null input only** — callers short-circuit and return an empty result. `Optional.of(value)` means **always proceed with the query**, regardless of whether `value` is a UUID token (HUMAN actor with a mapping) or the raw actorId (SYSTEM/AGENT actors, never tokenised). These two cases are intentionally conflated at call sites — both mean "search the ledger by this identifier." Custom implementations that return `Optional.empty()` to indicate "no token exists" will cause every query for untokenised actors to silently return empty, breaking SYSTEM and AGENT actor lookups without any error or log entry. The distinction between "token" and "raw actorId" can be made by comparing the returned value to the input — equal means no mapping, not-equal means a token was returned.
