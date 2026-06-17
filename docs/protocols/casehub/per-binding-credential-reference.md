---
id: PP-20260617-per-binding-credential-ref
title: "Per-binding credential reference: store a logical name in DB, resolve token from MicroProfile Config"
type: rule
scope: platform
applies_to: "Any casehub module that binds a per-channel or per-entity relationship to an external service requiring a credential, where different bindings may use different accounts/workspaces"
severity: important
refs:
  - ../../PLATFORM.md §Outbound Authentication
  - casehub/static-credentials-config-property-not-preferences.md
  - qhorus#261
created: 2026-06-17
---

## Context

The platform documents two credential tiers for outbound auth:

- **Tier 1 — Static deploy-time** (`@ConfigProperty`): one token per connector type, fixed at deploy time. Correct for connectors that address a single vendor account (Twilio SID, global Slack webhook URL).
- **Tier 2 — Named endpoint** (`EndpointRegistry` + `credentialRef`): multi-endpoint, per-tenant, resolved from a secrets backend. **`credentialRef` resolution is not yet implemented** — the field exists in `EndpointDescriptor` but nothing resolves it to an actual credential at runtime. Do not use `EndpointRegistry` as a secrets store today.

Neither tier fits the case where a single optional module needs **one credential per binding** (e.g. a different Slack bot token per Qhorus channel, one per Slack workspace), and the number of bindings is small and known at deploy time.

## Rule — Tier 1.5: Per-binding credential reference from MicroProfile Config

When a DB binding record needs to reference a credential that varies per binding:

1. **Store a logical name** (not the raw token) in the binding column: `credential_ref VARCHAR(128)`.
2. **Resolve at call time** via MicroProfile Config using a module-scoped key prefix:
   ```java
   Config config = ConfigProvider.getConfig();
   String token = config.getValue(
       "casehub.<module>.credentials." + credentialRef, String.class);
   ```
3. **Operators configure** each named credential as an env var or `application.properties` entry:
   ```
   casehub.qhorus.slack-channel.credentials.acme-workspace=xoxb-...
   casehub.qhorus.slack-channel.credentials.beta-workspace=xoxb-...
   ```

The logical name in the DB (`acme-workspace`) is the stable identifier. The actual token lives in the runtime environment, never in the DB. This follows the Serverless Workflow 1.0 `$secret: "name"` convention without requiring a secrets backend.

## Decision rule

| Situation | Pattern |
|-----------|---------|
| One credential per module type, fixed at deploy | Tier 1: `@ConfigProperty` |
| One credential per binding, ≤ dozens of bindings, deploy-time config | Tier 1.5: per-binding `credential_ref` + MicroProfile Config |
| Dynamic, per-tenant, runtime-registered, requires secrets backend | Tier 2: `EndpointRegistry` + `credentialRef` (when resolution is implemented) |

## What "credentialRef is deferred" means in PLATFORM.md

The `EndpointRegistry` SPI's `EndpointDescriptor.credentialRef` field (shipped platform#73) is a forward-compatibility placeholder. No runtime resolver reads it — adding `credentialRef` to an endpoint descriptor today has no effect. Until a secrets backend resolver is implemented (Vault integration, k8s Secret CSI, or similar), Tier 2 is unavailable. Tier 1.5 is the correct pattern for per-binding credentials in the interim.

## Reference implementation

`casehub-qhorus-slack-channel` module — `SlackBotBinding.credentialRef` + `SlackChannelBackend.resolveToken()`.

## Anti-patterns

- Storing raw tokens in DB binding columns — tokens in plaintext in the application DB.
- Using `PreferenceProvider` for credentials — `Preferences` is for runtime-mutable, path-scoped config, not deploy-time secrets. See `static-credentials-config-property-not-preferences.md`.
- Relying on `EndpointRegistry.credentialRef` for actual resolution today — the resolver is not implemented.
