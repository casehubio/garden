---
id: PP-20260527-d95fed
title: "Claudony SSE behavioral timing parameters must be PreferenceKey<T>, not ClaudonyConfig properties"
type: rule
scope: repo
applies_to: "claudony-app — any SSE behavioral timing parameter that the frontend observes or that varies by deployment"
severity: guidance
refs:
  - app/src/main/java/io/casehub/claudony/server/ChannelCursorStaleness.java
  - app/src/main/java/io/casehub/claudony/server/ChannelHeartbeatInterval.java
  - docs/protocols/casehub/typed-preference-keys.md
violation_hint: "A new timing constant added to ClaudonyConfig rather than as a SingleValuePreference record with a PreferenceKey — it won't appear in /api/mesh/config and can't be observed by the frontend or overridden via preference injection in tests"
created: 2026-05-27
---

`ClaudonyConfig` (via `@ConfigMapping`) is for deployment-time server configuration —
ports, modes, credentials, and static defaults that operators set once. SSE behavioral
timing parameters that the browser needs to observe (e.g. cursor staleness threshold,
heartbeat interval) belong in the platform preferences tier: a `SingleValuePreference`
record carrying a `PreferenceKey<T>` with namespace `"claudony"`, a camelCase name, and a
`defaultValue`. The value is resolved per-request via `PreferenceProvider`, exposed in
`MeshConfig` via `/api/mesh/config`, and overridable in tests via
`casehub.platform.preferences.defaults.claudony.<name>` without restarting the application.
Existing examples: `ChannelCursorStaleness` and `ChannelHeartbeatInterval`.
