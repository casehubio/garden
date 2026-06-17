---
id: PP-20260617-8de879
title: "ChatModel adapters must throw UnsupportedFeatureException for unsupported ResponseFormat types"
type: rule
scope: platform
applies_to: "Any LangChain4j ChatModel or StreamingChatModel adapter in casehub-platform or application repos"
severity: important
refs:
  - docs/protocols/casehub/ai-agent-provider-cdi-priority.md
violation_hint: "Adapter calls session.query() or proceeds past format check without throwing — caller receives an empty or malformed response with no error"
created: 2026-06-17
---

Any `ChatModel` or `StreamingChatModel` adapter that does not support a requested `ResponseFormat` type must call `request.responseFormat()` early in `doChat()` and throw `dev.langchain4j.exception.UnsupportedFeatureException` if the format is unsupported — before opening any session, subscribing to any stream, or routing to `handler.onError()`. This is a synchronous, programming-error-class throw: it fires from `doChat()` and propagates through `ChatModel.chat()` to the caller. Silent ignore is wrong — the caller believes JSON enforcement is active when it is not, producing unparseable output and silent data loss. The check must be the first statement in `doChat()` so that no session slot is consumed on an invalid request.
