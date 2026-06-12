---
id: PP-20260612-a2ef10
title: "ActionRiskClassifier implementations require @RiskClassifier CDI qualifier for engine discovery"
type: rule
scope: platform
applies_to: "Any application repo implementing ActionRiskClassifier"
severity: critical
refs:
  - GE-20260607-3b6711
  - GE-20260607-c39651
violation_hint: "Classifier implements ActionRiskClassifier and is @ApplicationScoped but silently has no effect — all classification decisions return Autonomous"
created: 2026-06-12
---

Consumer implementations of `ActionRiskClassifier` must be annotated with both `@RiskClassifier @ApplicationScoped`. The `@RiskClassifier` qualifier (`io.casehub.api.spi.RiskClassifier`) is required because `ChainedReactiveActionRiskClassifier` in the engine runtime discovers classifiers via `@Inject @RiskClassifier Instance<ActionRiskClassifier>`. Without the qualifier, the bean is invisible to this injection point and silently excluded from all classification decisions.

## Correct pattern

```java
@RiskClassifier @ApplicationScoped
public class AmlActionRiskClassifier implements ActionRiskClassifier {
    @Override public RiskDecision classify(PlannedAction action) { ... }
}
```

## Injection points must also carry the qualifier

Direct injection in `@QuarkusTest` must match:

```java
@Inject @RiskClassifier AmlActionRiskClassifier classifier;
```

Omitting `@RiskClassifier` on the injection point causes CDI resolution failure — the bean's qualifier set no longer matches `@Default`.

## Silent failure mode

A bean annotated `@ApplicationScoped` without `@RiskClassifier` compiles, deploys, and runs with zero errors. All `PlannedAction` declarations from workers are classified as `Autonomous` (no gate) because the chained classifier sees an empty `Instance<ActionRiskClassifier>`. The only observable symptom is that oversight gates never fire.
