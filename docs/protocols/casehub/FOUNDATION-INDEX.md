# CaseHub Foundation Protocols

For LLMs building the CaseHub platform itself — foundation modules, SPIs, and extensions
(casehub-engine, casehub-ledger, casehub-work, casehub-qhorus, casehub-connectors, parent).

**Building an app on CaseHub?** Read [HARNESS-INDEX.md](HARNESS-INDEX.md) instead.

Reconstitute this index: `grep -rl "^scope: platform\|^scope: repo" docs/protocols/casehub/ docs/protocols/universal/`

---

## Protocols

| File | Rule | Applies to |
|------|------|------------|
| [ai-agent-provider-cdi-priority.md](casehub/ai-agent-provider-cdi-priority.md) | AI agent domain SPIs: LangChain4j @DefaultBean in main module, Claude @ApplicationScoped in separate -claude/ module | casehub apps with domain SPIs that have LangChain4j + Claude Agent SDK implementations |
| [claude-agent-provider-vs-tmux-session-choice.md](casehub/claude-agent-provider-vs-tmux-session-choice.md) | Use ClaudeAgentProvider for ephemeral task-scoped invocations; tmux provisioners for persistent dashboard-visible sessions | casehub-platform-agent-claude consumers, claudony WorkerProvisioner implementations |
| [external-api-surface-in-deep-dive.md](casehub/external-api-surface-in-deep-dive.md) | Document new API surface in the repo deep-dive, not DESIGN.md | All casehub peer repos when adding new public types, SPIs, or services |
| [ci-dispatch-covers-direct-consumers.md](casehub/ci-dispatch-covers-direct-consumers.md) | CI dispatch chain must cover all direct Maven consumers — omitting one causes silent CI failure | all casehubio repos — every Trigger downstream CI step |
| [openclaw-hook-typescript-only.md](casehub/openclaw-hook-typescript-only.md) | OpenClaw before_prompt_build hooks require TypeScript Plugin SDK — Python App SDK has no hook registration | casehub-openclaw plugin/ directory |
| [no-workarounds-fix-the-design.md](casehub/no-workarounds-fix-the-design.md) | Fix the design — no workarounds, wrappers, or backward-compatibility shims; break callers and fix them | All design and implementation work across CaseHub modules |
| [otel-traceid-capture-before-fire-async.md](casehub/otel-traceid-capture-before-fire-async.md) | Capture OTel trace ID synchronously before CDI fireAsync() — never in the @ObservesAsync handler | Any casehub module firing CDI events via fireAsync() where OTel trace correlation is needed |
| [auth-retrofit-readiness.md](casehub/auth-retrofit-readiness.md) | No auth in domain/service; thin REST resources; injectable query filters; auth-free SPI signatures | All casehubio repos |
| [no-conditional-tenancy-filtering.md](casehub/no-conditional-tenancy-filtering.md) | tenancyId filtering must always execute unconditionally — never gate on deployment mode or feature flag | All modules — queries, events, cache keys, audit entries |
| [tenancy-repository-pattern.md](casehub/tenancy-repository-pattern.md) | Bind tenancyId inside data access classes — never at call sites | All modules — Repository pattern / data access layer |
| [tenancyid-server-side-only.md](casehub/tenancyid-server-side-only.md) | tenancyId is server-side infrastructure — never expose in client-facing APIs | All modules — REST DTOs, JSON schemas, OpenAPI specs, MCP tool parameters |
| [persistence-hibernate-tenant-aware-repository.md](casehub/persistence-hibernate-tenant-aware-repository.md) | All JPA repositories in casehub-persistence-hibernate must extend TenantAwareRepository | casehub-persistence-hibernate — all reactive JPA repository classes |
| [subcase-tenancyid-inherits-parent.md](casehub/subcase-tenancyid-inherits-parent.md) | SubCase tenancyId must be inherited from parent CaseInstance, never from currentPrincipal | casehub-engine — SubCaseExecutionHandler |
| [subcase-coordination-strategy.md](casehub/subcase-coordination-strategy.md) | Native M-of-N counting for simple thresholds; quarkus-flow for conditional/sequential orchestration; always behind SPI | casehub-engine blackboard |
| [flyway-migration-rules.md](universal/flyway-migration-rules.md) | Flyway namespace ranges, H2 mode, PostgreSQL testing | All modules with Flyway |
| [arc42stories-source-breadth.md](casehub/arc42stories-source-breadth.md) | Read all blogs, DESIGN.md, DESIGN-capabilities.md, ADRs, and specs before writing ARC42STORIES.MD — CLAUDE.md alone produces errors | Any session writing or updating ARC42STORIES.MD for a casehub module |
| [workspace-git-isolation.md](casehub/workspace-git-isolation.md) | Each project workspace must be its own isolated git repo — never a subdirectory of the parent workspace | All casehub project workspaces |
| [flyway-version-range-allocation.md](casehub/flyway-version-range-allocation.md) | Each module owns an exclusive Flyway thousand-block version range | All casehub modules using Flyway |
| [qhorus-flyway-consumer-versioning.md](casehub/qhorus-flyway-consumer-versioning.md) | qhorus consumer migrations start at V2000 — casehub-ledger owns V1000–V1999 in the combined qhorus datasource namespace | db/qhorus/migration/ — all Flyway migration files in the qhorus named datasource |
| [optional-module-pattern.md](universal/optional-module-pattern.md) | Optional Jandex library module pattern | All optional feature modules |
| [quarkus-test-database.md](universal/quarkus-test-database.md) | Database configuration for @QuarkusTest suites | All modules with @QuarkusTest |
| [sweep-blocked-item-process.md](casehub/sweep-blocked-item-process.md) | Sweep items with blockers must be labeled, documented, and removed — never silently skipped | sweep branches — S/XS umbrella issues |
| [eidos-validator-constant-per-field.md](casehub/eidos-validator-constant-per-field.md) | Each AgentDescriptorValidator field must have its own named constant — no reuse across semantically distinct fields | AgentDescriptorValidator in casehub-eidos-api |
| [library-jars-require-jandex.md](casehub/library-jars-require-jandex.md) | Every casehub library JAR that ships CDI beans or SPI supertypes must include a Jandex index | all casehub library modules — CDI beans and pure-SPI/interface JARs whose types are implemented by CDI beans |
| [casehub-engine-flow-module-isolation.md](casehub/casehub-engine-flow-module-isolation.md) | casehub-engine-flow depends only on casehub-engine-common — not on casehub-engine runtime | casehub-engine-flow and future optional engine extension modules |
| [spi-placement-caseinstance-goes-in-common.md](casehub/spi-placement-caseinstance-goes-in-common.md) | SPIs using CaseInstance or common/internal types must go in common/spi/, not api/spi/ | All SPI interfaces in casehub-engine modules |\n| [casehub-engine-flow-module-isolation.md](casehub/casehub-engine-flow-module-isolation.md) | casehub-engine-flow depends only on casehub-engine-common — not on casehub-engine runtime | casehub-engine-flow and future optional engine extension modules |
| [configmapping-prefix-ownership.md](casehub/configmapping-prefix-ownership.md) | All config keys under a @ConfigMapping prefix must be declared in the mapping interface | any Quarkus module using @ConfigMapping — casehub-platform-scim, casehub-work, and any future module with a config interface |
| [maven-module-scoping.md](universal/maven-module-scoping.md) | Always specify `-pl <module>` when running Maven commands | All multi-module casehub modules |
| [maven-submodule-folder-naming.md](universal/maven-submodule-folder-naming.md) | Submodule folder names are short — no repo prefix; `api`, `runtime`, `deployment` etc. | All multi-module casehub repos |
| [module-tier-structure.md](universal/module-tier-structure.md) | Three-tier module structure — pure-Java SPI / core library (no JPA) / full extension; no SDK types in SPI signatures | All casehubio multi-module repos |
| [connector-service-caller-api.md](casehub/connector-service-caller-api.md) | Inject ConnectorService, not Instance&lt;Connector&gt; | any casehub module that depends on casehub-connectors-core |
| [quartz-ram-store-configuration.md](universal/quartz-ram-store-configuration.md) | Use Quartz RAM store — no JDBC store, no Quartz tables | All casehub modules using Quartz |
| [ledger-spi-propagation.md](casehub/ledger-spi-propagation.md) | When a LedgerEntryRepository SPI method is added, update all downstream implementations | casehub-work, casehub-qhorus, casehub-engine |
| [ledger-sync-async-parity.md](casehub/ledger-sync-async-parity.md) | ~~RETIRED~~ — superseded by reactive-blocking-tier-separation + reactive-service-build-gating | — |
| [reactive-service-build-gating.md](casehub/reactive-service-build-gating.md) | Reactive-tier service beans are separate @ApplicationScoped Reactive*Service beans, @IfBuildProperty gated, with direct reactive SPI injection | Any casehub extension or module exposing service beans with reactive variants |
| [qhorus-reactive-gating.md](casehub/qhorus-reactive-gating.md) | qhorus diverges from reactive-service-build-gating: ExcludedTypeBuildItem is silently skipped in Quarkus 3.32.2 — use @IfBuildProperty per reactive bean + @ConfigRoot(BUILD_TIME) to gate reliably; property must not appear in application.properties | casehub-qhorus runtime reactive beans and QhorusProcessor |
| [ledger-reactive-spi-shim.md](casehub/ledger-reactive-spi-shim.md) | Reactive repository SPIs ship no bundled JPA impl; provide @DefaultBean blocking test shim in test/java | Any new reactive repository SPI in casehub-ledger |
| [ledger-subclass-extension.md](casehub/ledger-subclass-extension.md) | Ledger subclass rules — JOINED inheritance, V1004+ consumer migrations, domain-agnostic leaf hash | Any repo adding a LedgerEntry JPA subclass |
| [ledger-sequence-cross-subtype-query.md](casehub/ledger-sequence-cross-subtype-query.md) | Ledger sequence numbers must use findLatestBySubjectId() — spans all JOINED subtypes; subtype-scoped queries produce duplicate sequence numbers | casehub-engine-ledger and any module adding a new LedgerEntry JOINED subclass |
| [spi-default-method-contract-test.md](universal/spi-default-method-contract-test.md) | Verify SPI default method contracts with an anonymous implementation test — compiler error is the RED state | All SPI interfaces using default methods |
| [qhorus-event-content-null.md](casehub/qhorus-event-content-null.md) | EVENT message `content` is always null — render telemetry fields instead | All projects reading Qhorus ledger entries |
| [qhorus-human-governance-channel-types.md](casehub/qhorus-human-governance-channel-types.md) | Oversight channel must have `allowedTypes=QUERY,COMMAND`; human actors never post EVENT | All projects using Qhorus NormativeChannelLayout |
| [qhorus-actor-type-mapping.md](casehub/qhorus-actor-type-mapping.md) | All ActorType values must map to the canonical casehub ledger vocabulary | All casehubio projects assigning ActorType |
| [gateway-backend-registration-ordering.md](casehub/gateway-backend-registration-ordering.md) | Call open() before registerBackend() when registering a ChannelBackend | casehub-qhorus: any code calling ChannelGateway.registerBackend() |
| [gateway-backend-idempotent-registration.md](casehub/gateway-backend-idempotent-registration.md) | Always call deregisterBackend() before registerBackend() for non-human_participating backends to prevent duplicate fanOut() delivery | Any code registering human_observer or agent backends |
| [claudony-module-boundary-no-app-to-casehub.md](casehub/claudony-module-boundary-no-app-to-casehub.md) | claudony-casehub must not depend on claudony-app — app-layer beans must not be injected into casehub-layer beans | claudony-casehub module |
| [claudony-sse-timing-preference-key.md](casehub/claudony-sse-timing-preference-key.md) | Claudony SSE behavioral timing parameters must be PreferenceKey<T> exposed via /api/mesh/config — not ClaudonyConfig properties | claudony-app — any SSE behavioral timing parameter the frontend observes |
| [maven-coordinate-standard.md](universal/maven-coordinate-standard.md) | Maven coordinate standard for all casehubio repos | All casehubio Maven repos |
| [artifact-rename-propagation.md](universal/artifact-rename-propagation.md) | Artifact rename propagation — update all cross-repo consumers before shipping | Any casehubio repo renaming a published artifactId |
| [java-optional-usage.md](universal/java-optional-usage.md) | Use Optional only when absence is the method's primary return contract | All Java code across casehub |
| [quarkus-test-security-http-only.md](universal/quarkus-test-security-http-only.md) | Only add @TestSecurity to @QuarkusTest classes that exercise HTTP endpoints | All modules with @QuarkusTest classes |
| [quarkus-optional-extension-dep.md](universal/quarkus-optional-extension-dep.md) | Gate optional Quarkus extension deps via @IfBuildProperty on natural datasource property, not ExcludedTypeBuildItem | Quarkus extension runtime and deployment modules |
| [engine-spi-noops-defaultbean.md](casehub/engine-spi-noops-defaultbean.md) | Engine SPI no-op defaults must use @DefaultBean — bare @ApplicationScoped collides with consumer implementations | casehub-engine runtime no-op SPI beans |
| [engine-library-alternative-subclass-defaultbean.md](casehub/engine-library-alternative-subclass-defaultbean.md) | Concrete engine classes extending library @Alternative beans must use @DefaultBean — CDI does not inherit @Alternative; bare @ApplicationScoped subclass is always active | casehub-engine-ledger; any engine module extending a library @Alternative class |
| [case-definition-layers.md](casehub/case-definition-layers.md) | Three-layer case definition architecture — YAML (structure) + `*CaseDescriptor` POJO (business logic); `*CaseDefinitions` FuncDSL companions superseded for new harnesses; inherited from Serverless Workflow 1.0; do not collapse layers or bypass CaseDefinitionYamlMapper | casehub-engine; all CaseHub domain applications |
| [worker-function-execution-model.md](casehub/worker-function-execution-model.md) | Worker functions must use `FuncWorkflowBuilder.workflow().tasks(...).build()` — never raw lambdas in production; choose FuncDSL task type by operation (function/agent/get) | casehub-engine; all CaseHub domain applications |
| [typed-preference-keys.md](casehub/typed-preference-keys.md) | Use typed PreferenceKey<T> for SPI configuration — never stringly-typed get(String, Class<?>) | All casehubio SPI configuration and preference resolution |
| [spi-deletion-default-throws.md](casehub/spi-deletion-default-throws.md) | SPI defaults for data-deletion operations must throw UnsupportedOperationException, not silently no-op | All casehub-platform-api SPI interfaces with erasure or deletion methods |
| [casememorystore-adapter-asserttenant-contract.md](casehub/casememorystore-adapter-asserttenant-contract.md) | CaseMemoryStore adapters must call MemoryPermissions.assertTenant() at the top of every operation | Any class implementing CaseMemoryStore or ReactiveCaseMemoryStore |
| [platform-spi-contract.md](casehub/platform-spi-contract.md) | Platform SPI contract — mock scope, primary/secondary backend CDI pattern, Preference DEFAULT constant | All repos implementing casehub-platform-api SPIs |
| [spi-adapter-module-placement.md](universal/spi-adapter-module-placement.md) | SPI adapters start in the host repo as modules — extract to standalone repo only on confirmed cross-project adoption | Any new SPI adapter whose consumers at design time are all within CaseHub |
| [persistence-backend-cdi-priority.md](universal/persistence-backend-cdi-priority.md) | Three-tier CDI priority ladder — `@DefaultBean` → `@ApplicationScoped` → `@Alternative @Priority(1)` — backend activates by classpath presence, no consumer changes | All casehub modules implementing PreferenceProvider, WorkItemStore, AuditEntryStore, or any casehub persistence SPI |
| [platform-cdi-index-dependency.md](casehub/platform-cdi-index-dependency.md) | casehub-platform CDI modules must be added to quarkus.index-dependency in every consuming test config | Any module with @QuarkusTest injecting a bean from a casehub-platform artifact |
| [platform-testing-fixed-principal-setup.md](casehub/platform-testing-fixed-principal-setup.md) | @QuarkusTest modules using casehub-platform-testing must inject FixedCurrentPrincipal and call setTenancyId in @BeforeEach | Any @QuarkusTest class in a module with casehub-platform-testing on test classpath |
| [current-principal-boolean-delegates-to-actor-type.md](casehub/current-principal-boolean-delegates-to-actor-type.md) | CurrentPrincipal boolean classifier methods must delegate to actorType() — never independent actorId string matching | casehub-platform-api — CurrentPrincipal interface and all implementors |
| [peer-session-cross-repo-commit-discipline.md](casehub/peer-session-cross-repo-commit-discipline.md) | Peer repo Claude sessions must file issues on parent rather than committing directly — preserves branch scaffolding and issue traceability | All casehub peer repo Claude sessions |
| [spi-case-id-parameter.md](casehub/spi-case-id-parameter.md) | Strategy SPI methods must pass `caseId` as an explicit parameter — not captured at construction — so a future `PerCaseDynamicStrategy` can dispatch per-case without call-site changes | All repos defining strategy SPIs: engine, claudony, qhorus |
| [spi-reactive-blocking-io.md](casehub/spi-reactive-blocking-io.md) | Strategy SPIs whose implementations may perform blocking I/O must return `Uni<T>` from initial definition — retrofitting sync to reactive is a breaking multi-repo change with no compile-time signal | All casehub repos defining strategy SPIs — engine, claudony, qhorus, future modules |
| [yaml-humantask-binding-type.md](casehub/yaml-humantask-binding-type.md) | Use humanTask binding type for WorkItem-backed human gates — not capability | All harnesses writing YAML case definitions with human gates |
| [hitl-runtime-assembly.md](casehub/hitl-runtime-assembly.md) | HITL runtime requires casehub-engine-work-adapter + blackboard — explicit opt-in | All harnesses using humanTask YAML bindings |
| [work-adapter-test-subcase-group-repository.md](casehub/work-adapter-test-subcase-group-repository.md) | work-adapter @QuarkusTest requires MemorySubCaseGroupRepository in selected-alternatives | casehub-engine-work-adapter test module |
| [work-adapter-plan-item-running-ordering.md](casehub/work-adapter-plan-item-running-ordering.md) | PlanItem must not be marked RUNNING until all resolution and validation steps succeed | casehub-engine-work-adapter outbound handlers |
| [plan-item-store-blocking-transactional-call-site.md](casehub/plan-item-store-blocking-transactional-call-site.md) | PlanItemStore.save() must be called from a blocking @Transactional context | casehub-engine-work-adapter handlers writing PlanItem status |
| [work-adapter-inputmapping-payload-contract.md](casehub/work-adapter-inputmapping-payload-contract.md) | Engine adapters must propagate HumanTaskTarget inputMapping output to WorkItem payload | casehub-engine-work-adapter outbound handlers |
| [casehub-engine-inputdata-map-convention.md](casehub/casehub-engine-inputdata-map-convention.md) | Internal inputData is Map<String,Object> (post-evaluation); public entry points should accept Object per engine#302 | casehub-engine handlers, executors, schedulers |
| [claudony-reactive-spi-variants.md](casehub/claudony-reactive-spi-variants.md) | Claudony SPI implementations must use Reactive* variants — blocking variants throw BlockingOperationNotAllowedException on IO thread | claudony-casehub — WorkerContextProvider, WorkerProvisioner, CaseChannelProvider |
| [qhorus-consumer-integration-pattern.md](casehub/qhorus-consumer-integration-pattern.md) | Consumer service code must use `QhorusDashboardService` for dashboard views, not `ReactiveQhorusMcpTools` or raw entity services | Any REST resource or service in a consumer repo reading/writing Qhorus channels, instances, or messages |
| [external-content-modernised-blackboard.md](casehub/external-content-modernised-blackboard.md) | Use "Modernised Blackboard Architecture" in external-facing content — not "CMMN" | All external-facing content — website, landing pages, READMEs, blog posts, marketing materials |
| [no-negative-peer-project-references.md](casehub/no-negative-peer-project-references.md) | Do not reference peer projects negatively — describe what the code does, not what it avoids | All external-facing content — website, landing pages, READMEs, blog posts, marketing materials |
| [case-channel-message-signal.md](casehub/case-channel-message-signal.md) | `channelMessage` context path convention — payload shape, commitment-resolving types, channel naming, overwrite semantics | casehub-engine harnesses receiving human messages on Qhorus channels |
| [cross-backend-aggregation-service.md](casehub/cross-backend-aggregation-service.md) | Cross-backend aggregation: isolate failure per source, always return 200 with a sources field, JOIN over N+1, document identity invariant | Any casehub module assembling a unified view from multiple independent backends |
| [spi-evolution-default-methods.md](casehub/spi-evolution-default-methods.md) | Add new optional SPI capabilities as Java interface default methods with safe no-op returns; always add to abstract contract test | All casehub SPIs with multiple implementations |

---

## Garden References

Generic Quarkus/Java knowledge not specific to CaseHub. These apply to harness builders too —
referenced here; [HARNESS-INDEX.md](HARNESS-INDEX.md) points back to this section.

### CDI / Transactions

| GE-ID | Title |
|---|---|
| GE-20260512-66d997 | Panache static methods bypass CDI @Alternative stores — returns empty results silently |
| GE-20260512-0fe012 | CDI fireAsync() inside @Transactional dispatches immediately — observer can run before commit |
| GE-20260512-6887c9 | @ObservesAsync + @Transactional on same method is unreliable — delegate to separate bean |
| GE-20260512-a9ad9f | Raw ExecutorService drops CDI context — @Transactional silently broken on background threads |
| GE-20260512-6d0c2b | BroadcastProcessor.onNext() throws BackPressureFailure when no subscribers are registered |

### Quarkus Testing

| GE-ID | Title |
|---|---|
| GE-20260512-47f92e | quarkus-junit5 is a relocation stub since Quarkus 3.31 — use quarkus-junit |
| GE-20260512-b3f32a | @IfBuildProperty/@UnlessBuildProperty evaluated at augmentation only — QuarkusTestProfile has no effect |
| GE-20260512-e552f7 | @ApplicationScoped bean state persists across @QuarkusTest classes — tests pass in isolation, fail in suite |
| GE-20260512-50b394 | Use @TestTransaction + unique identifiers to prevent @Scheduled interference in tests |
| GE-20260512-c246b0 | Test CDI SPI with @Alternative static inner classes — Mockito cannot be injected as CDI beans |
| GE-20260512-493c90 | @QuarkusTest classes named *IT.java silently report 0 tests — failsafe collects them, not surefire |
| GE-20260512-c30f52 | @QuarkusIntegrationTest in runtime module causes class loading failures — separate module required |
| GE-20260513-3c1a03 | @TestSecurity silently ignored on @QuarkusTest classes that never touch HTTP |
| GE-20260513-4c4205 | Use AtomicInteger call counter in Supplier<String> to distinguish SSE events by content in tests |

### Database / Schema / Migrations

| GE-ID | Title |
|---|---|
| GE-20260512-ea776c | Quarkus named persistence units silently skip schema generation — explicit config per named PU |
| GE-20260512-a3838e | Transitive hibernate-reactive-panache causes H2 test startup failure — disable reactive datasource |
| GE-20260512-7720ab | H2-reserved words as column names pass PostgreSQL but fail in H2 test mode |
| GE-20260512-2c2eff | Non-ANSI SQL types in Flyway migrations pass H2 silently but fail on PostgreSQL at deployment |
| GE-20260512-67b3b5 | Panache find() alias-prefixed field names return empty results silently — bare field names required |

### Scheduler / Config

| GE-ID | Title |
|---|---|
| GE-20260512-1fa51e | @Scheduled interval without $ prefix silently fires at wrong frequency |
| GE-20260512-552405 | @ConfigMapping methods without Javadoc cause a compile error — not a runtime warning |
| GE-20260512-523f68 | Quarkus dev mode hot-reload silently breaks WebSocket endpoint registration |

### Architecture / Design Techniques

| GE-ID | Title |
|---|---|
| GE-20260512-e3e525 | OCC + policyTriggered flag for M-of-N threshold completion — prevents duplicate trigger under READ COMMITTED |
| GE-20260512-a09bd3 | Enforce blocking/reactive SPI method parity with a reflection test |

### Tooling

| GE-ID | Title |
|---|---|
| GE-20260512-a28ecc | Maven relative paths resolve to wrong worktree when shell cwd changes — use absolute paths |
| [epic-close-code-merge-required.md](epic-close-code-merge-required.md) | Code must be merged to main before EPIC-CLOSED.md or issue close | All casehub repos |
| [journal-migration-number-sync.md](journal-migration-number-sync.md) | Update journal migration V references if Flyway versions renumbered before journal is committed | All casehub repos |
| [builtin-strategy-registration.md](builtin-strategy-registration.md) | Register new built-in WorkerSelectionStrategy in activeStrategy() switch, @Alternative filter, and WorkItemsConfig Javadoc atomically | casehub-work |
| [epic-closed-md-deletion-date.md](epic-closed-md-deletion-date.md) | EPIC-CLOSED.md must include deletion date (today + 14 days) | Workspace + project epic branches |
| [quarkus-test-stub-reactive-cdi.md](quarkus-test-stub-reactive-cdi.md) | Unsatisfied reactive CDI deps in @QuarkusTest: use @DefaultBean stub in test sources | All casehub extension modules with non-reactive H2 test datasource |
| [qhorus-message-observer-scope.md](qhorus-message-observer-scope.md) | MessageObserver implementations must be @ApplicationScoped | casehub-qhorus |
| [blackboard-registry-call-order.md](casehub/blackboard-registry-call-order.md) | Call BlackboardRegistry.getOrCreate() before markConfigured() or indexForCompletion() | casehub-blackboard |
| [jq-evaluation-canonical.md](jq-evaluation-canonical.md) | JQ evaluation must go through JQEvaluator — never hand-rolled or direct JsonQuery | All casehub repos |
| [plan-item-delegated-state.md](casehub/plan-item-delegated-state.md) | Use markDelegated() for handlers that pass control to an external actor (SubCase, HumanTask, Extension) | casehub-blackboard, casehub-work-adapter |
| [plan-item-handler-error-path.md](casehub/plan-item-handler-error-path.md) | Outbound handlers must fault the PlanItem on error — silent return leaves PlanItem stuck | casehub-blackboard |
| [repo-hook-requirements.md](repo-hook-requirements.md) | All casehub repos with Work Tracking must have both pre-push and commit-msg hooks in .githooks/ | All casehub repos with Issue tracking: enabled |
| [issue-scale-complexity-labels.md](issue-scale-complexity-labels.md) | Every GitHub issue must carry scale: * and complexity: * labels at creation | All casehub repos |
| [work-event-type-enum-coverage.md](work-event-type-enum-coverage.md) | Every new WorkItemService audit event string must have a matching WorkEventType enum value in the same commit | casehub-work |
| [engine-cdi-event-await-chain.md](engine-cdi-event-await-chain.md) | @ConsumeEvent handlers fire CaseLifecycleEvent via .chain(completionStage()) not .invoke() | casehub-engine runtime |
| [provisioner-spi-provision-result.md](provisioner-spi-provision-result.md) | WorkerProvisioner.provision() returns ProvisionResult not Worker | casehub-engine api |
| [platform-mock-bean-exclusion.md](platform-mock-bean-exclusion.md) | Exclude casehub-platform @DefaultBean mocks in any module test config that also indexes casehub-persistence-memory | Any casehub engine module with both deps |
| [quarkus-maven-plugin-required.md](casehub/quarkus-maven-plugin-required.md) | Every module with @QuarkusTest must have quarkus-maven-plugin with generate-code goals | All casehub engine modules with @QuarkusTest classes |
| [delivery-audit-raw-output.md](delivery-audit-raw-output.md) | Delivery endpoints must pass raw agent output to Qhorus COMMAND — stripped content is for human interfaces only | casehub-openclaw: delivery endpoints dispatching to Qhorus |
| [disc-vocab-is-disposition.md](disc-vocab-is-disposition.md) | DISC types use dispositionVocabulary, not slotVocabulary — treating DISC as a role assignment is a category error | casehub-eidos-vocab producers; AgentDescriptor construction |
| [delegation-platform-semantic.md](delegation-platform-semantic.md) | delegation=true is a platform capability claim (can spawn sub-agents), not a personality trait — vocabulary producers must not set it | AgentDescriptor construction; personality vocabulary producers |
| [new-peer-repo-checklist.md](new-peer-repo-checklist.md) | New casehub peer repo must have CLAUDE.md, pre-push hook, CI/CD workflow, parent peer list entry, PLATFORM.md entries, and workspace repo before first commit | Any session creating a new casehub peer repo |
| [agent-descriptor-compact-constructor-validation.md](casehub/agent-descriptor-compact-constructor-validation.md) | AgentDescriptor required-field validation belongs in the compact constructor | AgentDescriptor record in casehub-eidos-api |
| [agent-routing-strategy-guard-before-emiton.md](casehub/agent-routing-strategy-guard-before-emiton.md) | AgentRoutingStrategy implementations must pre-screen guards before emitOn(workerPool) | Any AgentRoutingStrategy implementation dispatching to a worker pool via emitOn() |
| [alternative-extension-patterns.md](casehub/alternative-extension-patterns.md) | @Alternative Extension Patterns — two CDI override patterns for different contexts | Any casehubio repo extending a persistence SPI or overriding a default implementation |
| [arc42stories-foundation-tier-layer-taxonomy.md](casehub/arc42stories-foundation-tier-layer-taxonomy.md) | Foundation-tier ARC42STORIES.MD defines its own layer taxonomy — do not apply the harness layer sequence | Any foundation-tier module (casehub-ledger, casehub-work, casehub-qhorus, etc.) writing ARC42STORIES.MD |
| [auto-channel-key-sanitisation.md](casehub/auto-channel-key-sanitisation.md) | AutoChannelPolicy implementations must sanitise external keys via sanitiseSegment() | casehub-qhorus connector-backend — any AutoChannelPolicy embedding external identifiers in channel names |
| [blackboard-registry-get-blocking-required.md](casehub/blackboard-registry-get-blocking-required.md) | BlackboardRegistry.get() callers in @ConsumeEvent must use blocking = true | casehub-engine-blackboard — any @ConsumeEvent handler that calls BlackboardRegistry.get() |
| [bridge-module-spi-placement.md](casehub/bridge-module-spi-placement.md) | Bridge-module SPIs referencing bridge-internal types live in the bridge module, not api/spi/ | Any casehub bridge module (connector-backend, engine-ledger, etc.) |
| [casehub-platform-dependency-scope.md](casehub/casehub-platform-dependency-scope.md) | Use test scope for casehub-platform in library/extension modules; runtime scope in app modules | Any module declaring a dependency on io.casehub:casehub-platform |
| [cdi-registry-validate-names-at-startup.md](casehub/cdi-registry-validate-names-at-startup.md) | CDI self-registering strategy registries must validate member identity at construction time | Any casehubio CDI bean collecting @Any Instance<T> at startup and looking up members by string name |
| [channel-backend-idempotent-registration.md](casehub/channel-backend-idempotent-registration.md) | ChannelBackend implementations must deregister before registering in ChannelInitialisedEvent observer | Any class implementing ChannelBackend and observing ChannelInitialisedEvent |
| [channel-type-constraint-record-constructor.md](casehub/channel-type-constraint-record-constructor.md) | All channel type constraints must be validated in ChannelCreateRequest's compact constructor | casehub-qhorus — any code path creating a channel with type restrictions |
| [channel-type-policy-invariant.md](casehub/channel-type-policy-invariant.md) | allowedTypes and deniedTypes are architectural invariants — only set when a hard constraint is required | Any CaseChannelLayout implementation or code creating Qhorus channels |
| [claude-md-size-discipline.md](casehub/claude-md-size-discipline.md) | Keep CLAUDE.md under 25 KB — extract triggered content to referenced files | All casehubio repos with a CLAUDE.md |
| [claudony-channel-backend-registration-path.md](casehub/claudony-channel-backend-registration-path.md) | ClaudonyChannelBackend registration must go through ChannelInitialisedEvent | claudony-app; any code that registers ClaudonyChannelBackend in ChannelGateway |
| [consumer-spi-placement.md](casehub/consumer-spi-placement.md) | Consumer-Facing SPI Placement — api/spi/ vs runtime/ | Any casehubio extension or library module defining an SPI interface |
| [cross-foundation-bridge-module-placement.md](casehub/cross-foundation-bridge-module-placement.md) | Cross-foundation bridge modules live in the event-source repo as an optional submodule | casehub foundation repos when two peer modules need opt-in integration wiring |
| [cross-repo-issues-in-parent.md](casehub/cross-repo-issues-in-parent.md) | casehubio/parent issues are for coordinating simultaneous execution across 2+ repos | Any issue requiring changes spanning multiple casehubio repos |
| [cross-repo-optional-dep-table-registration.md](casehub/cross-repo-optional-dep-table-registration.md) | Register optional cross-repo dependencies in both the build order and the dependency table | All casehubio repos when adding an optional cross-repo compile dependency |
| [dual-trail-audit-pattern.md](casehub/dual-trail-audit-pattern.md) | Dual-Trail Audit Pattern — operational trail + compliance ledger written independently | casehub-engine, casehub-work, and any future module writing both an operational trail and a compliance ledger |
| [eidos-enrichment-constants-in-pipeline.md](casehub/eidos-enrichment-constants-in-pipeline.md) | Enrichment step constants shared between blocking and reactive paths belong in EidosRenderPipeline | Any new enrichment step added to casehub-eidos |
| [eidos-poc-gate-before-jpa-infrastructure.md](casehub/eidos-poc-gate-before-jpa-infrastructure.md) | casehub-eidos: validate new capabilities with in-memory PoC before JPA infrastructure | casehub-eidos — any new persistence-backed capability |
| [eidos-spi-agent-scope-requires-tenancy-id.md](casehub/eidos-spi-agent-scope-requires-tenancy-id.md) | casehub-eidos SPI methods scoped to an agent must include tenancyId | casehub-eidos-api SPI interfaces; any new SPI method identifying or querying a specific agent |
| [engine-api-scope-rule.md](casehub/engine-api-scope-rule.md) | Depend on casehub-engine-api (not casehub-engine) in modules that implement engine SPIs without running the engine | Any module implementing WorkerProvisioner, CaseChannelProvider, or WorkerStatusListener SPIs |
| [engine-ledger-migration-if-not-exists.md](casehub/engine-ledger-migration-if-not-exists.md) | Engine-ledger Flyway migrations adding columns to shared tables must use IF NOT EXISTS | casehub-engine-ledger — db/engine-ledger/migration/ |
| [eval-dimension-format-only-gate.md](casehub/eval-dimension-format-only-gate.md) | EvalDimension.applicableFor() is gated by format only — never by profile availability | casehub-eidos eval module; any code adding a new EvalDimension |
| [eval-variant-pair-single-axis-isolation.md](casehub/eval-variant-pair-single-axis-isolation.md) | Eval variant pairs must differ on exactly one AgentDisposition axis | casehub-eidos eval module; any AgentProfile variant pair in profiles/index.yaml |
| [event-log-left-fold-projection.md](casehub/event-log-left-fold-projection.md) | Event-log left-fold projection — derive channel read-models as a pure fold | Any casehubio consumer building a read-model or digest from Qhorus channel message history |
| [fleet-channel-notify-no-qhorus-dispatch.md](casehub/fleet-channel-notify-no-qhorus-dispatch.md) | Fleet channel notify endpoint must call ChannelEventBus.emit() directly — never re-enter Qhorus dispatch | claudony-app; any fleet endpoint handling cross-node channel message delivery |
| [fleet-client-filter-registration.md](casehub/fleet-client-filter-registration.md) | Fleet HTTP clients must register FleetKeyClientFilter on RestClientBuilder, not via @RegisterProvider | claudony-app; any code building a PeerClient via RestClientBuilder.newBuilder() |
| [fleet-key-role-separation.md](casehub/fleet-key-role-separation.md) | Fleet key must grant fleet role, not user role | claudony-app; ApiKeyAuthMechanism |
| [flyway-data-migration-naming-dependency.md](casehub/flyway-data-migration-naming-dependency.md) | Flyway data migrations targeting rows by naming convention must document the coupling | casehub-qhorus — any Flyway migration using UPDATE ... WHERE name LIKE on channel/entity names |
| [flyway-ledger-migration-locations.md](casehub/flyway-ledger-migration-locations.md) | Include db/ledger/migration in Flyway locations for any module consuming casehub-ledger | Any module whose test classpath transitively includes casehub-ledger |
| [format-specific-enrichment-schema-isolation.md](casehub/format-specific-enrichment-schema-isolation.md) | Format-specific LLM enrichment concerns belong in a dedicated step with their own schema | Any casehub renderer running an LLM enrichment step shared across multiple output formats |
| [inbound-connector-id-is-type-not-account.md](casehub/inbound-connector-id-is-type-not-account.md) | InboundMessage.connectorId must be the connector type string — never account- or instance-level | casehub-connectors — all InboundConnector and WebhookInboundConnector implementations |
| [inbound-connector-numeric-metadata-always-present.md](casehub/inbound-connector-numeric-metadata-always-present.md) | Numeric InboundMessage metadata keys must always be present — never absent when zero | casehub-connectors — all implementations writing numeric metadata |
| [inbound-connector-type-separation.md](casehub/inbound-connector-type-separation.md) | Pull-based and webhook-based inbound connectors must use distinct types — no unified SPI | casehub-connectors — any module implementing inbound message transport |
| [inmemory-store-aggregate-no-scan-delegation.md](casehub/inmemory-store-aggregate-no-scan-delegation.md) | InMemoryStore aggregate methods must not delegate to scan() — stream the backing collection directly | casehub-qhorus-testing — InMemory*Store implementations of count(), sum(), or any aggregate method |
| [jta-tsr-status-active-gate.md](casehub/jta-tsr-status-active-gate.md) | Check STATUS_ACTIVE before registering JTA synchronizations | casehub-qhorus — any @Transactional service deferring post-commit side effects via TSR |
| [ledger-algorithm-transparent-signing.md](casehub/ledger-algorithm-transparent-signing.md) | casehub-ledger signing and verification code must be algorithm-transparent | io.casehub.ledger.runtime.service — any class that signs, verifies, or loads cryptographic keys |
| [ledger-enricher-priority-mandate.md](casehub/ledger-enricher-priority-mandate.md) | All LedgerEntryEnricher implementations must declare @Priority | casehub-ledger — any class implementing LedgerEntryEnricher |
| [ledger-identity-cache-rotation-invalidation.md](casehub/ledger-identity-cache-rotation-invalidation.md) | Per-actorId identity caches in casehub-ledger must observe AgentKeyRotatedEvent for invalidation | casehub-ledger — any CDI bean or AbstractCachingIdentityProvider subclass caching identity data keyed by actorId |
| [ledger-test-profile-datasource.md](casehub/ledger-test-profile-datasource.md) | QuarkusTestProfile restart requires casehub.ledger.datasource in getConfigOverrides() when ledger JPA is used | Any @QuarkusTest class whose @TestProfile causes a Quarkus context restart with casehub-ledger JPA beans |
| [ledger-write-api-transaction-demarcation.md](casehub/ledger-write-api-transaction-demarcation.md) | casehub-ledger write API outer methods must not be @Transactional — delegate to a separate @Transactional service | casehub-ledger — any combined write API |
| [llm-pass-structural-fallback.md](casehub/llm-pass-structural-fallback.md) | Foundation extension LLM pass must have a format-specific structural fallback | Any casehub Foundation extension adding an optional LLM enrichment step |
| [local-spi-placeholder-contract-identity.md](casehub/local-spi-placeholder-contract-identity.md) | Local SPI placeholders must have a contract identical to the planned engine-api SPI | casehub integration repos when defining a local interface for later promotion to casehub-engine-api |
| [mcp-tool-exception-catch-all.md](casehub/mcp-tool-exception-catch-all.md) | MCP tool methods must catch Exception broadly and return 'Failed: ...' — never propagate | Any @Tool-annotated method on a Quarkus MCP server exposed to LLM agents |
| [memory-storeall-transactional-contract.md](casehub/memory-storeall-transactional-contract.md) | CaseMemoryStore storeAll() must guarantee atomicity — single-transaction for JDBC, pre-flight assertTenant for REST | All CaseMemoryStore adapter implementations (jpa, sqlite, inmem, mem0) |
| [memory-erase-by-id-completeness.md](casehub/memory-erase-by-id-completeness.md) | ERASE_BY_ID capability requires complete erasure — source record plus all derived data | Any CaseMemoryStore adapter declaring MemoryCapability.ERASE_BY_ID |
| [message-dispatch-builder-validation.md](casehub/message-dispatch-builder-validation.md) | Speech-act type validation is owned by MessageDispatch.Builder — enforce at build(), not downstream | casehub-qhorus — any code dispatching messages via MessageService.dispatch() |
| [message-service-dispatch-enforcement-gate.md](casehub/message-service-dispatch-enforcement-gate.md) | MessageService.dispatch() is the single enforcement gate for all channel writes — no caller may bypass it | casehub-qhorus — any code sending a message to a Qhorus channel |
| [openclaw-delivery-always-200.md](casehub/openclaw-delivery-always-200.md) | OpenClaw delivery endpoints always return 200 — never let the agent runtime retry | casehub-openclaw — any @Path('/openclaw/delivery/*') or '/openclaw/plugin/*' endpoint |
| [platform-cache-domain-event-bridge.md](casehub/platform-cache-domain-event-bridge.md) | Platform beans that cannot observe domain events get a thin domain-side bridge observer | Any casehub platform bean with a TTL cache that must be invalidated by a domain event |
| [platform-ownership-check.md](casehub/platform-ownership-check.md) | Platform ownership check — pause before implementing infrastructure in a domain repo | Any casehubio domain repo implementing a new class or module |
| [plugin-hooks-call-rest-not-mcp.md](casehub/plugin-hooks-call-rest-not-mcp.md) | Plugin hooks call Quarkus REST directly — MCP endpoint is LLM-facing only | casehub-openclaw plugin/ module; any extension adding plugin hooks |
| [protocol-refs-use-double-dotdot.md](casehub/protocol-refs-use-double-dotdot.md) | Protocol refs from docs/protocols/casehub/ use ../../ to reach docs/ level | Any protocol file in docs/protocols/casehub/ with a refs: entry pointing to docs/ |
| [publish-workflow-repository-dispatch.md](casehub/publish-workflow-repository-dispatch.md) | publish.yml must include repository_dispatch trigger for upstream-published events | Every casehubio repo with a publish.yml workflow |
| [python-component-in-maven-repo.md](casehub/python-component-in-maven-repo.md) | Python components in Maven repos live in python/ with own pyproject.toml — not as Maven modules | Integration-tier repos (casehub-openclaw and future repos mixing Java and Python) |
| [qhorus-channel-dual-identity.md](casehub/qhorus-channel-dual-identity.md) | Qhorus channels have dual identity: immutable UUID + immutable name | casehub-qhorus — MCP tools, service layer, any consumer creating or referencing channels |
| [qhorus-entity-mapper-pure-transformer.md](casehub/qhorus-entity-mapper-pure-transformer.md) | QhorusEntityMapper methods must not inject stores or issue queries — all data arrives as parameters | casehub-qhorus — QhorusEntityMapper and any future mapper class |
| [qhorus-service-store-seam.md](casehub/qhorus-service-store-seam.md) | Qhorus service classes must query through store interfaces — never Panache static calls | casehub-qhorus — any @ApplicationScoped service needing data from Channel, Message, or Commitment entities |
| [reactive-mcp-tool-resolve-channel-blocking.md](casehub/reactive-mcp-tool-resolve-channel-blocking.md) | Reactive @Tool methods that call resolveChannel() must carry @Blocking | casehub-qhorus — ReactiveQhorusMcpTools and any future reactive MCP tool class |
| [reactive-pg-devservices-test-profile.md](casehub/reactive-pg-devservices-test-profile.md) | Use Quarkus Dev Services named-datasource profile for reactive PostgreSQL integration tests | Any casehubio module with a named datasource adding reactive PostgreSQL integration tests |
| [reactive-render-pipeline-threading.md](casehub/reactive-render-pipeline-threading.md) | Reactive eidos renderers must follow the three-stage threading pattern | Any reactive SystemPromptRenderer or ReactiveSystemPromptRenderer implementation in casehub-eidos |
| [reactive-rendered-prompt-cache-canonical-spi.md](casehub/reactive-rendered-prompt-cache-canonical-spi.md) | Both eidos renderers must inject ReactiveRenderedPromptCache — not RenderedPromptCache | EidosSystemPromptRenderer, DefaultReactiveSystemPromptRenderer, any future renderer in casehub-eidos |
| [reactive-spi-bridge-default-bean.md](casehub/reactive-spi-bridge-default-bean.md) | Reactive SPI bridge defaults must be @DefaultBean without @IfBuildProperty | Any casehub Foundation module adding a reactive SPI bridge implementation |
| [reactive-test-entity-setup-named-pu.md](casehub/reactive-test-entity-setup-named-pu.md) | Reactive @QuarkusTest entity setup for named-PU services: use QuarkusTransaction.requiringNew() with blocking service | @QuarkusTest integration tests for reactive services backed by a named Quarkus datasource |
| [render-format-structure-naming.md](casehub/render-format-structure-naming.md) | RenderFormat enum values must name output structure, not LLM provider | SystemPromptRenderer.RenderFormat in casehub-eidos-api; any future output format enum |
| [renderer-cache-key-includes-format.md](casehub/renderer-cache-key-includes-format.md) | Renderer cache keys must include the output format dimension | Any casehub component caching format-specific rendered output |
| [scim2-agent-identity-lookup.md](casehub/scim2-agent-identity-lookup.md) | Resolve agent identity attributes via SCIM2 Agent endpoint using actorId as externalId | Any casehub component resolving agent identity attributes — casehub-ledger, casehub-eidos, casehub-engine |
| [spi-event-tenancyid-component-order.md](casehub/spi-event-tenancyid-component-order.md) | CDI SPI event records carrying tenancyId must place it as the 2nd component after caseId | casehub-engine-common/spi/event/ — any new SPI event record that carries tenancyId |
| [spi-test-scope-default-bean-noop.md](casehub/spi-test-scope-default-bean-noop.md) | SPIs with only test-scoped implementations need a @DefaultBean no-op in the runtime module | Any casehub-work runtime module injecting a platform SPI with only test-scoped implementations |
| [static-credentials-config-property-not-preferences.md](casehub/static-credentials-config-property-not-preferences.md) | Use @ConfigProperty for static deploy-time credentials; use Preferences for runtime multi-tenant configuration | Any casehub module requiring external service credentials |
| [tmux-test-await-not-sleep.md](casehub/tmux-test-await-not-sleep.md) | Tmux pane state transitions in tests must use Await.until(), not Thread.sleep() | claudony-app; any @QuarkusTest creating a tmux session |
| [webhook-signature-constant-time.md](casehub/webhook-signature-constant-time.md) | All webhook HMAC signature verification must use MessageDigest.isEqual() — never String.equals() | Any casehub module verifying inbound webhook signatures |
| [workitem-template-constraints-snapshot-at-instantiation.md](casehub/workitem-template-constraints-snapshot-at-instantiation.md) | Snapshot template-defined constraints onto WorkItem at instantiation, never re-read at completion | casehub-work WorkItemService.create(), all instantiation paths |
| [eidos-vocabulary-metadata-required.md](eidos-vocabulary-metadata-required.md) | @VocabularyMetadata required on all VocabularyTerm enum classes — absent annotation causes runtime IllegalArgumentException at startup | casehub-eidos: any enum implementing VocabularyTerm; any VocabularyRegistrar |
| [eidos-axis-exact-match-exhaustive-switch.md](eidos-axis-exact-match-exhaustive-switch.md) | axisExactMatch must use exhaustive switch on DispositionAxis with no default branch | casehub-eidos: any VocabularyTerm overriding axisExactMatch |
| [eidos-vocabulary-registry-register-postconstruct-only.md](eidos-vocabulary-registry-register-postconstruct-only.md) | CdiVocabularyRegistry.register() is @PostConstruct-only — not thread-safe after startup | casehub-eidos: any caller of VocabularyRegistry.register() |
| [eidos-vocab-uri-for-axis-not-inlined.md](eidos-vocab-uri-for-axis-not-inlined.md) | Use vocabUriForAxis() — do not inline the disposition axis vocabulary resolution chain | casehub-eidos consumers resolving vocabulary URIs for AgentDescriptor disposition fields |
| [jpa-mapper-positional-constructor-field-completeness.md](jpa-mapper-positional-constructor-field-completeness.md) | Full-fidelity JPA-to-record mappers must use the positional constructor, not Builder — Builder silently defaults new fields to null | casehub-eidos AgentDescriptorMapper.toRecord(); any full-fidelity JPA→record mapper |
| [runtime-test-cdi-exclude-ledger-capture-beans.md](runtime-test-cdi-exclude-ledger-capture-beans.md) | Exclude CaseLedgerEventCapture/WorkerDecisionEventCapture from CDI when ledger Flyway migrations are absent — otherwise every case times out silently | casehub-engine runtime test profile; any module with casehub-engine-ledger as test dep |
| [case-started-payload-is-panel-document.md](case-started-payload-is-panel-document.md) | CASE_STARTED EventLog payloads must use panel document format {working:{...},semantic:{},episodic:{}} — flat maps cause IllegalArgumentException in fromPanelDocument() | Any test manually constructing CASE_STARTED EventLog entries |
