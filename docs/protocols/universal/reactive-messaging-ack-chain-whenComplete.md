---
id: PP-20260623-a0fe15
title: "Use .whenComplete() not .exceptionally() when fireAsync() is chained to a SmallRye message ack"
type: rule
scope: universal
applies_to: "SmallRye Reactive Messaging @Incoming consumers that dispatch CDI events and chain .thenCompose(message.ack()) after fireAsync()"
severity: critical
refs:
  - jvm/GE-20260621-629712.md
violation_hint: "cloudEventBus.fireAsync(ce).exceptionally(ex -> { LOG.warn(...); return null; }).thenCompose(ignored -> message.ack()) — .exceptionally() swallows the exception, .thenCompose runs, message is acked even on dispatch failure"
garden_ref: "GE-20260621-629712"
created: 2026-06-23
---

In SmallRye Reactive Messaging `@Incoming` consumers, the `CompletionStage` returned by
`cloudEventBus.fireAsync()` must be handled with `.whenComplete()` — **not** `.exceptionally()`
— before `.thenCompose(message.ack())`. `.exceptionally()` swallows the exception and returns
`null`; the chained `.thenCompose()` then fires unconditionally and acks the message even when
dispatch failed, silently consuming the message with no processing. `.whenComplete()` preserves
the exceptional state; `.thenCompose()` skips its lambda; the exception propagates to SmallRye,
which nacks the message for retry or dead-letter routing. GE rule 3 (use `.exceptionally()` for
fire-and-forget CloudEvent dispatch) does NOT apply here — the stage is not fire-and-forget
when it is part of a transactional ack chain.
