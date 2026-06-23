# Universal Protocols

Reusable conventions for any Java/Quarkus project — not specific to CaseHub.
Mandated conventions for all projects in this ecosystem. When a Hortora-hosted
protocols repository exists, these files will move there and be replaced with references.

Reconstitute: `grep -rl "^scope: universal" docs/protocols/universal/*.md`

---

## Maven / Build

| File | Rule | Applies to |
|------|------|------------|
| [maven-coordinate-standard.md](maven-coordinate-standard.md) | Maven coordinate standard — groupId, artifactId, version conventions | Any Maven project |
| [maven-module-scoping.md](maven-module-scoping.md) | Always specify `-pl <module>` when running Maven commands | Any multi-module Maven project |
| [maven-submodule-folder-naming.md](maven-submodule-folder-naming.md) | Submodule folder names are short — no parent prefix; `api`, `runtime`, `deployment` | Any multi-module Maven project |
| [artifact-rename-propagation.md](artifact-rename-propagation.md) | Artifact rename propagation — update all consumers before shipping | Any multi-repo Maven project |

## Java / Libraries

| File | Rule | Applies to |
|------|------|------------|
| [filesystem-watching-library.md](filesystem-watching-library.md) | Use `io.methvin:directory-watcher` for filesystem watching — not raw `WatchService` | Any Java project detecting filesystem changes in a directory tree |
| [qdrant-client-library.md](qdrant-client-library.md) | Use `io.qdrant:client` directly for Qdrant — not `quarkus-langchain4j-qdrant` | Any Java project storing or querying embeddings in Qdrant |

## Java / API Design

| File | Rule | Applies to |
|------|------|------------|
| [varargs-type-capture.md](varargs-type-capture.md) | Consider varargs type capture over `Class<T>` when type flows only into a lambda — eliminates explicit class argument at call site | Any Java API accepting `Class<T>` solely for lambda type inference |

## Java / Architecture

| File | Rule | Applies to |
|------|------|------------|
| [java-optional-usage.md](java-optional-usage.md) | Use Optional only when absence is the method's primary return contract | Any Java project |
| [module-tier-structure.md](module-tier-structure.md) | Three-tier module structure — pure-Java SPI / core library (no JPA) / full extension | Any library or framework |
| [spi-adapter-module-placement.md](spi-adapter-module-placement.md) | SPI adapters start in the host repo as modules — extract to standalone repo only on confirmed cross-project adoption | Any multi-module Java project with SPIs and pluggable adapters |
| [optional-module-pattern.md](optional-module-pattern.md) | Optional Jandex library module pattern | Any library with optional features |
| [cdi-classpath-presence-requires-module-separation.md](cdi-classpath-presence-requires-module-separation.md) | @ApplicationScoped impl must be in a separate module for classpath-presence CDI activation to work | Any Quarkus project using @DefaultBean fallback displaced by optional @ApplicationScoped |
| [spi-default-method-contract-test.md](spi-default-method-contract-test.md) | Verify SPI default method contracts with an anonymous implementation test | Any library with SPIs and default methods |
| [spi-signature-change-all-impls-same-commit.md](spi-signature-change-all-impls-same-commit.md) | Update all in-repo SPI implementations when a method signature changes — in the same commit | Any Java project with multiple SPI implementations in one repo |

## Quarkus

| File | Rule | Applies to |
|------|------|------------|
| [quarkus-junit-not-junit5.md](quarkus-junit-not-junit5.md) | Use quarkus-junit not quarkus-junit5 in all new Quarkus modules | All Maven modules with quarkus-junit5 test dep |
| [quarkus-test-database.md](quarkus-test-database.md) | Database configuration for @QuarkusTest suites | Any Quarkus app with @QuarkusTest |
| [quarkus-test-security-http-only.md](quarkus-test-security-http-only.md) | Only add @TestSecurity to @QuarkusTest classes that exercise HTTP endpoints | Any Quarkus app with @TestSecurity |
| [quartz-ram-store-configuration.md](quartz-ram-store-configuration.md) | Use Quartz RAM store — no JDBC store, no Quartz tables | Any Quarkus app using Quartz |
| [quarkus-optional-extension-dep.md](quarkus-optional-extension-dep.md) | Gate optional Quarkus extension deps via Capabilities + ExcludedTypeBuildItem | Any Quarkus extension |
| [library-jar-annotation-only-deps.md](library-jar-annotation-only-deps.md) | Library JARs (no quarkus:build) must use annotation-only deps (e.g. micrometer-core), not Quarkus extensions (e.g. quarkus-micrometer) | Any Maven library JAR consumed by Quarkus apps |
| [flyway-migration-rules.md](flyway-migration-rules.md) | Flyway migration conventions — naming, H2 compatibility, PostgreSQL testing | Any project using Flyway |
| [entity-not-null-java-default-matches-sql-default.md](entity-not-null-java-default-matches-sql-default.md) | NOT NULL entity fields with a SQL DEFAULT must carry a matching Java-level field default — Hibernate drop-and-create omits SQL DEFAULTs, breaking tests that persist without setting the field | Any Hibernate/Panache project adding NOT NULL columns via ALTER TABLE migration |
| [flyway-repo-scoped-migration-path.md](flyway-repo-scoped-migration-path.md) | Flyway migrations must ship under a repo-scoped path — never the generic db/migration | All Quarkus modules shipping Flyway migrations |
| [quarkus-extension-flyway-locations-explicit.md](quarkus-extension-flyway-locations-explicit.md) | Extensions must not ship quarkus.flyway.locations — consumers configure migration paths explicitly | Any Quarkus extension with Flyway migrations |
| [quarkus-void-buildstep-produce-anchor.md](quarkus-void-buildstep-produce-anchor.md) | Void @BuildStep must use @Produce(ArtifactResultBuildItem.class) to guarantee execution | Any Quarkus extension with side-effect-only @BuildStep |

| [flyway-extension-migration-registration.md](flyway-extension-migration-registration.md) | Quarkus extensions must use repo-scoped Flyway migration paths and register SQL resources for native image | Any Quarkus extension shipping Flyway migrations |
| [quarkus-extension-unremovable-consumer-beans.md](quarkus-extension-unremovable-consumer-beans.md) | Quarkus extension CDI beans with no internal injection point must be annotated @Unremovable | Any Quarkus extension publishing a CDI bean that consumers inject but the extension never injects internally |

## CDI / Dependency Injection

| File | Rule | Applies to |
|------|------|------------|
| [persistence-backend-cdi-priority.md](persistence-backend-cdi-priority.md) | Three-tier CDI priority ladder — `@DefaultBean` → `@ApplicationScoped` → `@Alternative @Priority(1)` — backend activates by classpath presence | Any Quarkus project with multiple competing CDI implementations of the same persistence SPI |
| [startup-bean-config-self-contained.md](startup-bean-config-self-contained.md) | @Startup beans must read config via @ConfigProperty directly — no cross-bean @PostConstruct dependencies | Any @Startup @ApplicationScoped bean that depends on shared configuration or global state |

| [no-jpa-entities-across-requires-new.md](no-jpa-entities-across-requires-new.md) | No JPA entities cross a REQUIRES_NEW boundary — extract primitives before the call | Any Quarkus service calling a @Transactional(REQUIRES_NEW) method from within an outer @Transactional context |
| [reactive-messaging-ack-chain-whenComplete.md](reactive-messaging-ack-chain-whenComplete.md) | Use `.whenComplete()` not `.exceptionally()` when `fireAsync()` is chained to `message.ack()` — `.exceptionally()` silently acks on failure | Any SmallRye `@Incoming` consumer that dispatches CDI events and chains `.thenCompose(message.ack())` |

## REST / JAX-RS

| File | Rule | Applies to |
|------|------|------------|
| [jax-rs-provider-cdi-scope.md](jax-rs-provider-cdi-scope.md) | JAX-RS `@Provider` beans in Quarkus must carry an explicit CDI scope annotation (`@ApplicationScoped` for stateless mappers) | Any Quarkus project using JAX-RS `@Provider` beans |

| [sanitize-caller-controlled-headers-before-logging.md](sanitize-caller-controlled-headers-before-logging.md) | Sanitize caller-controlled HTTP headers before including them in logs | Any Quarkus service that logs HTTP request context — especially security event logs |

## CI / GitHub Actions

| File | Rule | Applies to |
|------|------|------------|
| [cross-org-repo-format-in-ci.md](cross-org-repo-format-in-ci.md) | Use org/repo format in CI REPOS lists when the ecosystem spans multiple GitHub orgs | Any CI workflow iterating over repos from more than one GitHub org |

## Integration / Cross-Module Contracts

| File | Rule | Applies to |
|------|------|------------|
| [opaque-cross-module-identifiers.md](opaque-cross-module-identifiers.md) | Opaque cross-module identifiers must be stored unchanged — never parsed by the receiver | Any module receiving an identifier whose format is owned by another module |
| [cross-repo-coordination-issues.md](cross-repo-coordination-issues.md) | Multi-repo coordination issues belong in the platform root repo only when simultaneous execution is required — not for single-repo implementations consumed by multiple repos | Any multi-repo platform with a root/coordination repo |

## Application Design

| File | Rule | Applies to |
|------|------|------------|
| [layer-log.md](layer-log.md) | Maintain LAYER-LOG.md as definition of done per adoption layer | Any layered application built on a platform |
| [reactive-blocking-tier-separation.md](reactive-blocking-tier-separation.md) | Service beans must not carry dependencies on capabilities optional in consuming deployments — blocking and reactive tiers are separate beans | Any extension library with heterogeneous deployment contexts |
| [reactive-vs-blocking-selection.md](reactive-vs-blocking-selection.md) | Choose reactive vs blocking based on I/O profile and concurrency model — never mix within a request path; persistence model must follow execution model | Any Quarkus/Vert.x-based module choosing an execution model |
| [committed-git-hooks.md](committed-git-hooks.md) | Commit git hooks to .githooks/ with core.hooksPath — never install to .git/hooks/ | All git repositories receiving shared hooks |
| [event-log-left-fold-projection.md](event-log-left-fold-projection.md) | Project append-only event-log history into derived views using a pure left-fold — deterministic, independently testable, cursor-resumable | Any component building a summary, digest, or read-model from a typed event log |
