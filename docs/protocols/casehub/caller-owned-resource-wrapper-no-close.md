---
id: PP-20260617-609995
title: "Plain-class wrappers around caller-supplied AutoCloseable resources must never call close()"
type: rule
scope: platform
applies_to: "Any plain class (no CDI) in casehub-platform that wraps an AutoCloseable supplied by the caller — AgentSessionChatModel, future resource wrappers"
severity: important
refs:
  - docs/protocols/universal/module-tier-structure.md
violation_hint: "Wrapper calls close() in a terminal handler or finally block on a resource it did not open — caller's resource is silently invalidated after the first use"
created: 2026-06-17
---

A plain-class adapter that wraps an `AutoCloseable` supplied by its constructor (e.g. `AgentSessionChatModel(AgentSession session)`) must never call `close()` on that resource — not in `finally` blocks, not in Mutiny terminal handlers (`onFailure`, `onCompletion`), not on any error path. Lifecycle ownership follows construction: the caller who supplies the resource is responsible for closing it. The wrapper's `doChat()` or streaming path may observe the resource fail and surface that failure to the caller, but it does not clean up what it didn't create. This rule prevents double-close bugs and semaphore slot leaks in cases where the caller uses try-with-resources for the same resource. Contrast with CDI-managed adapters (`ClaudeAgentChatModel`) which own their sessions and must close them on all paths.
