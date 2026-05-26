---
id: PP-20260526-case-channel-message-signal
title: "channelMessage signal path — convention for Qhorus human message bridging to case context"
type: rule
scope: platform
applies_to: "casehub-engine harnesses that deploy casehub-engine-work-adapter and receive human messages on Qhorus channels"
severity: important
refs:
  - engine#349
created: 2026-05-26
---

## Convention

When a human (or agent) sends a commitment-resolving Qhorus message on a case channel,
`QhorusMessageSignalBridge` writes the message payload to the case context under the path
`channelMessage`. Case definitions react to this via a `contextChange` trigger:

```yaml
bindings:
  - name: handle-human-response
    on:
      contextChange: ".channelMessage"
    when: ".channelMessage.purpose == \"oversight\""
    target:
      capability: processHumanInput
```

## Payload Shape

The value at `context["channelMessage"]` is a `Map<String, Object>` containing:

| Field | Type | Present when |
|-------|------|-------------|
| `messageType` | String | Always — RESPONSE, DONE, DECLINE, or FAILURE |
| `content` | String | Always (non-null for commitment-resolving types) |
| `senderId` | String | Always |
| `channelId` | String (UUID) | Always |
| `channelName` | String | Always — format `case-{caseId}/{purpose}` |
| `correlationId` | String | Only if non-null on the originating message |

## Commitment-Resolving Types

Only these four `MessageType` values trigger a signal: **RESPONSE, DONE, DECLINE, FAILURE**.
These are the types that resolve a Qhorus Commitment. COMMAND, QUERY, STATUS, EVENT, and
HANDOFF are intentionally excluded:
- EVENT has null content (PP-20260508-90428f) and is informational only
- COMMAND creates obligations but does not resolve them
- QUERY/STATUS are interrogative/informational
- HANDOFF transfers obligation; the receiving agent's outcome arrives later as DONE/FAILURE

## Channel Naming

Case channels follow the format `case-{caseId}/{purpose}` as defined by
`CaseChannel.CASE_CHANNEL_PREFIX` and `CaseChannel.channelName(UUID, String)` in
`casehub-engine-api`. The bridge uses `CaseChannel.CASE_CHANNEL_PREFIX` to identify case
channels. Any `CaseChannelProvider` implementation that creates channels for cases **must**
use this naming convention — otherwise the bridge cannot extract the `caseId` and will
silently ignore messages.

## Overwrite Semantics

Each signal overwrites the previous `channelMessage` value. Case definitions that need to
track a conversation history should query the Qhorus channel directly (via the channelId
in the payload) — the signal is a trigger, not a history store.

## When Signals Reach WAITING Cases

With `PlanningStrategyLoopControl` (blackboard active), a WAITING case processes
`CONTEXT_CHANGED` events. A human message on a case channel therefore unblocks a WAITING
case and triggers binding re-evaluation. Case definitions can react to the message
(e.g., cancel the waiting work) or let the case continue waiting.
