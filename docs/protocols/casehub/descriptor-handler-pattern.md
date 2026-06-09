---
id: PP-20260609-descriptor-handler-pattern
title: "Descriptor+Handler pattern — centralise enum behaviour in POJOs, not switch statements"
type: rule
scope: application
applies_to: "casehubio application repos (aml, clinical, devtown, life, quarkmind) — any repo where enum values have distinct behaviour (routing, SLA, capabilities, templates, worker logic)"
severity: required
refs:
  - casehub-life#27 — first full application of the pattern across all layers (11 locations across 6 service/observer classes)
created: 2026-06-09
---

# Protocol: Descriptor+Handler Pattern

**Applies to:** casehubio application repos — any repo where domain enum values carry distinct behaviour  
**Severity:** Required — switch statements scattered across service classes are a coherence violation detectable at review time

---

## The Pattern

Every enum whose values have distinct behaviour (routing policy, SLA, capabilities, template category, worker logic) must carry that behaviour in a **Descriptor** POJO, not in switch statements in service or observer classes.

### Descriptor — pure Java POJO

A `*CaseDescriptor` (or `*Descriptor` for non-case contexts) is a pure Java record or class that carries all declarative knowledge about a domain type:

- Worker lambdas (what the worker does)
- Capability tags (which agents can handle this type)
- Routing policy (which channel, which SLA tier)
- Template category (which template to use)
- SLA policy (deadline, escalation policy)
- Any other type-specific configuration

The descriptor is **tested independently of Quarkus** — no CDI, no database, no application context. If a descriptor test needs a framework, the logic belongs in a Handler instead.

```java
// Example from casehub-life#27
public record HouseholdActionDescriptor(
    HouseholdActionType type,
    String capabilityTag,
    SlaBreachPolicy slaPolicy,
    WorkItemTemplate template
) {
    // All behaviour for this action type lives here — not scattered across services
    public static HouseholdActionDescriptor forType(HouseholdActionType type) {
        return REGISTRY.get(type);
    }
}
```

### Handler — optional CDI supplement

A `*Handler` or `*CaseHandler` is an optional `@ApplicationScoped` CDI bean that adds execution behaviour requiring infrastructure (repositories, preference providers, connectors). It is only created when the descriptor alone is insufficient.

```java
@ApplicationScoped
public class HouseholdActionHandler {
    @Inject WorkItemService workItemService;
    @Inject PreferenceProvider preferences;

    public void execute(HouseholdActionDescriptor descriptor, ...) {
        // Execution logic using CDI infrastructure
    }
}
```

---

## The Rule

**In casehubio application repos: every enum whose values have distinct behaviour must carry that behaviour in its descriptor or handler — never in a switch statement in a service or observer class.**

This rule applies to all enum types that currently or will have per-value logic:
- Domain type enums (`LifeDomain`, `HouseholdActionType`, `LifeCaseType`, etc.)
- Commitment mode enums (`CommitmentMode`)
- Escalation tier enums
- Any other enum where `switch (value) { case A: doX(); case B: doY(); }` appears in a service

---

## The Coherence Check

Before implementing any feature or adding logic to a service class, ask:

> **"Am I adding a switch on an enum value in a service class?"**

If the answer is **yes** — the logic belongs in the enum's descriptor or handler instead.

Equally: before writing a `Map<EnumType, SomeBehaviour>` or `if (type == X) { ... } else if (type == Y) { ... }` in a service — stop. That belongs in the descriptor.

---

## Where Business Logic Lives

| Logic type | Location | Why |
|---|---|---|
| Worker lambda (what a worker does) | `*CaseDescriptor` | Pure Java, testable independently of Quarkus |
| Capability tag (which agents handle it) | `*CaseDescriptor` | Declarative — no runtime infrastructure needed |
| SLA policy | `*CaseDescriptor` | Policy is declarative; enforcement is infrastructure |
| Template selection | `*CaseDescriptor` | Declarative lookup |
| Execution using CDI beans (repos, connectors) | `*Handler` | Requires infrastructure — keep out of the POJO |
| Switch on enum type | ❌ NEVER in a service | Violates this protocol; belongs in descriptor |

---

## Reference Implementation

**casehub-life#27** is the first full application of this pattern across all layers. It found and eliminated 11 distinct switch/map/conditional locations across 6 service and observer classes. Use it as the concrete example when explaining or auditing compliance with this protocol.

Affected patterns in life#27:
- Static maps keyed on `LifeDomain` or `HouseholdActionType` → moved to descriptors
- `if (domain == X)` chains in service classes → moved to descriptors
- Per-type capability routing → moved to descriptors
- Per-type SLA policy selection → moved to descriptors

---

## Violation Examples

```java
// ❌ Violation — switch on enum in a service class
public class LifeCaseService {
    public String getCapabilityTag(LifeDomain domain) {
        return switch (domain) {
            case HOUSEHOLD -> "household-management";
            case HEALTH -> "health-monitoring";
            case FINANCE -> "financial-planning";
        };
    }
}

// ❌ Violation — static map in a service class
public class HouseholdActionService {
    private static final Map<HouseholdActionType, WorkItemTemplate> TEMPLATES = Map.of(
        HouseholdActionType.REPAIR, REPAIR_TEMPLATE,
        HouseholdActionType.CLEANING, CLEANING_TEMPLATE
    );
}
```

```java
// ✅ Correct — logic in descriptor
public record LifeDomainDescriptor(LifeDomain domain, String capabilityTag, ...) {
    public static LifeDomainDescriptor forDomain(LifeDomain domain) { ... }
}

// ✅ Correct — service is thin, delegates to descriptor
public class LifeCaseService {
    public String getCapabilityTag(LifeDomain domain) {
        return LifeDomainDescriptor.forDomain(domain).capabilityTag();
    }
}
```
