---
id: PP-20260529-7f6b73
title: "OpenClaw before_prompt_build integration requires TypeScript Plugin SDK — Python App SDK has no hook registration"
type: rule
scope: repo
applies_to: "casehub-openclaw plugin/ directory — any feature requiring an OpenClaw lifecycle hook"
severity: important
refs:
  - docs/specs/2026-05-29-epic5-python-sdk-design.md
  - docs/specs/openclaw-integration.md
violation_hint: "A Python file (context_hook.py, or any .py file) attempting to call agent.on() or register a before_prompt_build handler"
created: 2026-05-29
---

OpenClaw's `before_prompt_build` hook (and all lifecycle hooks) are only available in the OpenClaw TypeScript Plugin SDK — not the Python App SDK. The Python App SDK (`from openclaw import OpenClawClient`) is an external REST API wrapper with no `agent.on()` method and no in-process hook registration mechanism. Any casehub-openclaw feature requiring an OpenClaw lifecycle hook must ship a TypeScript entry point in `plugin/` with a synchronous `register(api: OpenClawPluginApi): void` function. Hooks are snapshotted by OpenClaw at registration time (~30s before the gateway starts serving traffic) — wiring hooks from the async `start()` lifecycle phase silently drops them. The Python package (`python/`) is a client library for explicit channel context queries from Python skill scripts; it owns no hook logic.
