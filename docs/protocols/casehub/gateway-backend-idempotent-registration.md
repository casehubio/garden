---
id: PP-20260522-4c3d86
title: "Always call deregisterBackend() before registerBackend() for non-human_participating backends"
type: rule
scope: platform
applies_to: "Any code that calls ChannelGateway.registerBackend() for human_observer or agent backends — e.g. ClaudonyChannelBackend registration in MeshResource and ServerStartup"
severity: important
refs:
  - app/src/main/java/io/casehub/claudony/server/MeshResource.java
  - app/src/main/java/io/casehub/claudony/server/ServerStartup.java
  - docs/protocols/casehub/gateway-backend-registration-ordering.md
violation_hint: "gateway.registerBackend() called without a preceding gateway.deregisterBackend() — can produce duplicate entries causing double-delivery on every fanOut()"
created: 2026-05-22
---

`ChannelGateway.registerBackend()` has no deduplication guard for `human_observer`
(or `agent`) backend types — only `human_participating` throws on duplicate. Calling
`registerBackend()` twice for the same channel and backend produces two entries;
`fanOut()` then calls `post()` twice per message, causing duplicate delivery. The
correct idempotent sequence is: `gateway.deregisterBackend(channelId, backendId)` (no-op
if absent) → `backend.open(ref, Map.of())` → `gateway.registerBackend(channelId, backend,
backendType)`. This applies everywhere a backend may be re-registered: SSE endpoint
on each new client connect, and `ServerStartup.bootstrapChannelBackends()` on server
restart. See also `gateway-backend-registration-ordering.md` for the `open()`-before-`register()`
ordering constraint.
