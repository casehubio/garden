---
id: PP-20260526-fe9b64
title: "Inject ConnectorService, not Instance<Connector>"
type: rule
scope: platform
applies_to: "any casehub module that depends on casehub-connectors-core"
severity: important
refs:
  - casehub-connectors/docs/DESIGN.md
violation_hint: "A module injecting Instance<Connector> directly and writing stream().filter().findFirst() routing"
created: 2026-05-26
---

Callers that need to send outbound messages must inject `ConnectorService`, not
`Instance<Connector>`. `ConnectorService` provides id-based routing, consistent
handling of unknown ids (throws `IllegalArgumentException` with available ids in
the message), duplicate-id detection at startup, and `supports()`/`ids()` for
capability checks. Routing directly via `Instance<Connector>` duplicates the
lookup pattern across callers, diverges on the not-found case, and leaks CDI
internals into consumer code.
