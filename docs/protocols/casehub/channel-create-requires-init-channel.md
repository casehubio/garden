---
id: PP-20260609-fe1300
title: "Callers of ChannelService.create() / ReactiveChannelService.create() must call gateway.initChannel() on the created channel"
type: rule
scope: platform
applies_to: "Any code that calls ChannelService.create() or ReactiveChannelService.create() and expects ChannelBackend implementations to receive messages on the created channel"
severity: critical
refs:
  - docs/protocols/casehub/gateway-backend-registration-ordering.md
garden_ref: "GE-20260609-9ee2ad"
violation_hint: "A newly created channel receives COMMAND messages but OpenClawChannelBackend (or any ChannelBackend) never handles them — ChannelGateway.fanOut() finds no registered backends for the channel because ChannelInitialisedEvent was never fired for it"
created: 2026-06-09
---

`ChannelService.create()` and `ReactiveChannelService.create()` persist the channel to the database but do not call `ChannelGateway.initChannel()` or fire `ChannelInitialisedEvent`. The startup hook (`ChannelGateway.onStart()`) fires `initChannel` for all channels already in the DB at boot; channels created after startup are not covered. Any `ChannelBackend` that registers via `@Observes ChannelInitialisedEvent` (e.g. `OpenClawChannelBackend`) will not be registered for the new channel, and `fanOut()` will silently skip it. Callers must call `gateway.initChannel(ch.id, new ChannelRef(ch.id, ch.name))` immediately after `create()` — on the create path only, not on the `findByName` path (existing channels are already registered from startup). In reactive chains, use `.invoke(ch -> gateway.initChannel(ch.id, new ChannelRef(ch.id, ch.name)))` inside the flatMap that creates the channel.
