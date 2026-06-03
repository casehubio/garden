---
id: PP-20260603-c36746
title: "AI agent domain SPIs use two-tier CDI priority: LangChain4j @DefaultBean, Claude @ApplicationScoped in separate module"
type: rule
scope: platform
applies_to: "Any casehub application domain SPI that has both a LangChain4j implementation (any LLM) and a Claude Agent SDK implementation (Claude-only)"
severity: important
refs:
  - docs/protocols/universal/optional-module-pattern.md
  - docs/protocols/universal/persistence-backend-cdi-priority.md
  - casehubio/platform#55
violation_hint: "Both implementations in the same Maven module defeats classpath-presence activation — @ApplicationScoped always wins over @DefaultBean regardless of classpath"
created: 2026-06-03
---

When an application domain SPI (e.g. `DebateAgentProvider`) has both a LangChain4j default implementation (any LLM, portable, works in CI) and a Claude Agent SDK alternative (Claude-only, requires CLI), structure the CDI activation as a two-tier classpath-presence pattern: the LangChain4j implementation uses `@DefaultBean @ApplicationScoped` in the main application module; the Claude implementation uses `@ApplicationScoped` (no `@DefaultBean`) in a **separate Maven module** (e.g. `drafthouse-claude/`) that declares `casehub-platform-agent-claude` as a compile dependency. Adding the separate module to the application's POM activates the Claude implementation; omitting it leaves LangChain4j active. The Claude implementation injects `AgentProvider` (the platform SPI), not `ClaudeAgentClient` directly. Both implementations must pass the domain SPI's parity contract tests; reasoning quality is out of CI scope.
