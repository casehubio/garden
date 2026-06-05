---
id: PP-20260605-175a00
title: "Delivery endpoint audit records must preserve raw agent output — not stripped content"
type: rule
scope: platform
applies_to: "casehub-openclaw: any delivery endpoint that receives agent output and dispatches to Qhorus"
severity: important
refs:
  - casehubio/openclaw#10
violation_hint: "Delivery endpoint passes speechAct.content() (stripped) as the Qhorus COMMAND message content instead of the raw output parameter — audit record omits speech-act classification metadata"
created: 2026-06-05
---

Any delivery endpoint that receives OpenClaw agent output and dispatches to Qhorus must preserve the raw agent output in the Qhorus COMMAND message — the durable audit record. Stripped or processed content (bracket prefixes removed, JSON envelope extracted) is only appropriate for human-facing interfaces such as oversight prompts.

The rule separates two concerns with different consumers:

- **Audit record** (Qhorus COMMAND message `.content()`): machine-retrievable, raw — what the agent actually produced
- **Human interface** (oversight prompt, display): stripped — the intended action without classification metadata

Reference implementation — `OversightGateService.openGate()` (casehubio/openclaw#10):

```java
// COMMAND content = raw output for audit fidelity (machine-retrievable)
messageService.dispatch(MessageDispatch.builder()
    .content(rawOutput != null ? rawOutput : "")  // raw
    ...);
// Human oversight prompt uses stripped content
String oversightPrompt = buildOversightPrompt(agentId, speechAct.content(), gate);
```
