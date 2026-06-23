---
id: PP-20260623-105mcp
title: "Library MCP tools must use @McpServer with a named server"
type: rule
scope: platform
applies_to: "Any Quarkus library (extension or runtime JAR) that exposes @Tool-annotated methods via quarkus-mcp-server — casehub-qhorus, casehub-connectors-mcp, and any future MCP surface"
severity: important
refs:
  - casehubio/claudony#105
  - casehubio/qhorus#306
  - docs/specs/2026-04-13-quarkus-ai-ecosystem-design.md
violation_hint: "A library's @Tool class has no @McpServer annotation — tools land on the default server, polluting the application's tool list and risking pagination collisions when another library is also embedded."
created: 2026-06-23
---

The default MCP server belongs to the application. Libraries that expose `@Tool` methods must annotate their tool classes with `@McpServer("<library-name>")` and ship default config via `META-INF/microprofile-config.properties` in the runtime JAR:

```properties
quarkus.mcp.server.<name>.http.root-path=/<name>
quarkus.mcp.server.<name>.server-info.name=<name>
quarkus.mcp.server.<name>.tools.page-size=0
```

`microprofile-config.properties` has ordinal 100 (below application.properties at 250), so consumers can override the path. Without the default config, `invalidServerNameStrategy` (default: `FAIL`) causes a startup failure — the tools reference a server that doesn't exist.

The `tools.page-size=0` disables the 50-tool default pagination cap on the library's server. This is the library's responsibility — consumers should not need to know how many tools the library has.

This convention mirrors named datasources (`quarkus.datasource.qhorus.*`) and named Flyway (`quarkus.flyway.qhorus.*`). It prevents tool-list pollution, pagination collisions, and audience mismatch when multiple libraries are embedded. Reference implementation: casehub-qhorus (qhorus#306).
