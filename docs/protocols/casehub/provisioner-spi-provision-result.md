---
id: PP-20260529-bcbbb5
title: "WorkerProvisioner.provision() returns ProvisionResult, not Worker"
type: rule
scope: repo
applies_to: "casehub-engine — WorkerProvisioner and ReactiveWorkerProvisioner SPIs and all their implementations"
severity: critical
refs:
  - engine#389 (implementation)
  - api/src/main/java/io/casehub/api/spi/ProvisionResult.java
violation_hint: "provision() method signature returning Worker or Uni<Worker> instead of ProvisionResult or Uni<ProvisionResult>"
created: 2026-05-29
---

`WorkerProvisioner.provision()` and `ReactiveWorkerProvisioner.provision()` MUST return `ProvisionResult` (blocking) and `Uni<ProvisionResult>` (reactive) respectively. `Worker` is a case-definition artifact — its purpose is to describe a worker's capabilities, function, and execution policy as declared by the case author. Provisioning outcomes (e.g. causal ledger entry linkage via `causedByEntryId`) are not part of the definition and must not be grafted onto `Worker`. Provisioner implementations that do not resolve a causal entry should return `ProvisionResult.empty()`. No-op defaults throw `ProvisioningException` unchanged.
