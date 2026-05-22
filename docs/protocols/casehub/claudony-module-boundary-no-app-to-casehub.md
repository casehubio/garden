---
id: PP-20260522-c741d7
title: "claudony-casehub must not depend on claudony-app — app-layer beans must not be injected into casehub-layer beans"
type: rule
scope: repo
applies_to: "claudony — claudony-casehub module (SPI implementations)"
severity: critical
refs:
  - app/src/main/java/io/casehub/claudony/server/ClaudonyChannelBackend.java
  - app/src/main/java/io/casehub/claudony/server/MeshResource.java
  - casehub/src/main/java/io/casehub/claudony/casehub/ClaudonyReactiveCaseChannelProvider.java
violation_hint: "Injecting ClaudonyChannelBackend, ChannelEventBus, or any other @ApplicationScoped bean from claudony-app into a claudony-casehub bean"
created: 2026-05-22
---

`claudony-app` depends on `claudony-casehub`; the reverse creates a circular Maven
dependency that fails at build time. Beans that implement Qhorus or platform SPIs and
require app-layer infrastructure (e.g. `ClaudonyChannelBackend`, `ChannelGateway`) must
live in `claudony-app`, not in `claudony-casehub`. When a casehub-layer SPI needs to
trigger an app-layer concern — such as registering a channel backend when a channel is
opened — use CDI async events (following the `WorkerCaseLifecycleEvent` pattern) or
delegate registration entirely to the app layer (e.g. register on SSE subscribe in
`MeshResource`), rather than injecting app beans into casehub beans.
