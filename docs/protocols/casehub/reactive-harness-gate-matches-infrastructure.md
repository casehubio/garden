---
id: PP-20260609-49ebf4
title: "Integration-tier reactive SPI implementations must use the same @IfBuildProperty gate as their qhorus reactive dependencies"
type: rule
scope: platform
applies_to: "Any casehub integration-tier harness (casehub-openclaw, claudony, any future harness) implementing ReactiveWorkerProvisioner, ReactiveCaseChannelProvider, or other reactive SPIs that inject ReactiveChannelService or ReactiveMessageService"
severity: critical
refs:
  - docs/protocols/casehub/reactive-service-build-gating.md
garden_ref: "GE-20260609-9ee2ad"
violation_hint: "A reactive SPI bean is gated on a separate flag (e.g. casehub.openclaw.reactive.enabled) while its injected dependencies (ReactiveChannelService, ReactiveMessageService) are gated on casehub.qhorus.reactive.enabled — Quarkus startup fails with UnsatisfiedResolutionException when only the harness flag is set"
created: 2026-06-09
---

When a reactive SPI implementation injects `ReactiveChannelService` or `ReactiveMessageService` (which are themselves gated on `casehub.qhorus.reactive.enabled=true`), the implementation bean must use the same gate — `@IfBuildProperty(name="casehub.qhorus.reactive.enabled", stringValue="true")`. Introducing a separate harness-specific flag (e.g. `casehub.openclaw.reactive.enabled`) creates a two-flag co-deployment requirement: if only the harness flag is set, the bean activates but its injected reactive services are absent, causing `UnsatisfiedResolutionException` at startup. A single gate is the correct design: activating qhorus reactive implies all harnesses depending on it should also activate. See the openclaw reactive SPI implementation (`ReactiveOpenClawWorkerProvisioner`, `ReactiveOpenClawCaseChannelProvider`) as the reference — both gated directly on `casehub.qhorus.reactive.enabled`. Refs PP-20260519-39a9a5 (reactive-service-build-gating).
