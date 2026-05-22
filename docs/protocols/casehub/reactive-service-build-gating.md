---
id: PP-20260519-39a9a5
title: "Reactive-tier service beans in casehub extensions are separate beans, build-time gated, with direct reactive SPI injection"
type: rule
scope: platform
applies_to: "Any casehub extension or module that exposes service beans with reactive variants (e.g. casehub-ledger, casehub-qhorus)"
severity: important
refs:
  - docs/protocols/universal/reactive-blocking-tier-separation.md
violation_hint: "A single @ApplicationScoped bean mixes blocking and reactive methods, or a reactive bean uses Instance<T> guards instead of build-time gating, or a reactive bean is not suffixed Reactive*Service"
enforcement: "casehub-ledger BlockingReactiveParityTest (ArchUnit 1.4.1) — auto-discovers Reactive*Service classes by naming convention, asserts bidirectional method parity and Uni<T> return types"
created: 2026-05-19
updated: 2026-05-22
---

Implements PP-20260519-f2e160 (reactive-blocking-tier-separation) for the Quarkus/casehub
platform. Every service capability ships in two separate @ApplicationScoped beans: a
blocking-tier bean (no reactive imports, no Instance<T> wrappers) and a reactive-tier bean
(Reactive*Service suffix, direct @Inject of reactive SPIs, all methods return Uni<T>).

## Build-Time Gating (Quarkus Extensions)

For Quarkus **extensions** (with runtime/ and deployment/ modules), the reactive tier is
gated via `@IfBuildProperty` on each reactive bean, backed by a `@ConfigRoot(phase =
BUILD_TIME)` config interface in the deployment module. This is the **canonical and preferred
approach** — used by `casehub-qhorus` (the reference implementation).

**Canonical pattern (casehub-qhorus):**

```java
// deployment/ — declares the build-time property formally
@ConfigMapping(prefix = "casehub.qhorus")
@ConfigRoot(phase = ConfigPhase.BUILD_TIME)
public interface QhorusBuildTimeConfig {
    ReactiveConfig reactive();
    interface ReactiveConfig {
        @WithDefault("false")
        boolean enabled();
    }
}

// runtime/ — reactive beans gated on the build property
@IfBuildProperty(name = "casehub.qhorus.reactive.enabled", stringValue = "true")
@ApplicationScoped
public class ReactiveJpaChannelStore implements ReactiveChannelStore {
    @Inject ChannelReactivePanacheRepo repo;  // also @IfBuildProperty-gated
    @WithTransaction
    public Uni<Channel> put(Channel channel) { return repo.persist(channel); }
}
```

`@IfBuildProperty` on runtime beans is only reliable in extensions when the property is
formally declared as `BUILD_TIME` phase config — without it, the property may not be
evaluated at augmentation time. `ExcludedTypeBuildItem` in `@BuildStep` is an equivalent
alternative but more verbose; prefer `@IfBuildProperty` + BUILD_TIME config.

For Quarkus **applications** (no deployment module), `@IfBuildProperty` is acceptable
directly on the bean if the property is declared BUILD_TIME.

Consuming deployments that need reactive set `casehub.<module>.reactive.enabled=true` in
`application.properties` at build time. Test suites set it in `test/resources/application.properties`.

## Transaction Management

Reactive JPA implementations use `@WithTransaction` (from
`io.quarkus.hibernate.reactive.panache.common`) on write methods. This CDI interceptor
handles Vert.x context setup automatically — no manual `withSafeContext` wrapper needed
when using `quarkus-hibernate-reactive-panache`.

The `withSafeContext` / `AbstractJpaRepository` pattern (casehub-engine) is required only
when calling `Panache.withSession()` or `Panache.withTransaction()` as static methods.
Prefer `@WithTransaction` (annotation-based) over static Panache methods.

## Parity Rule

Adding a method to the blocking tier requires adding the Uni<T> equivalent to the reactive
tier, and vice versa — parity is structural, not co-located.

This parity rule is enforced at build time in `casehub-ledger` via `BlockingReactiveParityTest`
(ArchUnit 1.4.1), which auto-discovers all `Reactive*Service` classes by naming convention and
asserts bidirectional method parity and `Uni<T>` return types. A vacuous-pass guard
(`assertThat(count).isGreaterThanOrEqualTo(1)`) ensures the rule cannot silently pass when no
classes are matched.
