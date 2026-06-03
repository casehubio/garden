---
id: PP-20260603-a25235
title: "Use ClaudeAgentProvider for ephemeral task-scoped invocations; tmux provisioners for persistent dashboard-visible sessions"
type: rule
scope: platform
applies_to: "casehub-platform-agent-claude consumers, claudony WorkerProvisioner implementations, any module launching Claude CLI as a worker"
severity: guidance
refs:
  - casehubio/platform#57
violation_hint: "Adding persistent session tracking to ClaudeAgentProvider, or using a tmux session for one-shot task execution that has no dashboard visibility requirement"
created: 2026-06-03
---

`ClaudeAgentProvider` and tmux-based provisioners (e.g. `ClaudonyReactiveWorkerProvisioner`) both launch Claude CLI but serve distinct architectural roles. Use `ClaudeAgentProvider` when the consumer needs ephemeral, task-scoped execution: one `run(AgentSessionConfig)` call, streamed `Multi<AgentEvent>` output, wall-clock timeout, no state after completion — the right fit for casehub-engine WorkerProvisioner implementations. Use a tmux-based provisioner when the session must be persistent (survives the provisioning call), visible in the Claudony dashboard, externally terminatable by workerId, and wired into the ledger causal context chain (`causedByEntryId` via `causalContext` map). The two patterns are complementary: migrating Claudony's dashboard-visible workers to `ClaudeAgentProvider` v1 would lose persistence, dashboard visibility, and external termination — all of which are load-bearing in the current architecture.
