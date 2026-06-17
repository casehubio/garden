# CaseHub Agentic Harness Protocols

For LLMs building applications on top of CaseHub — domain apps, living labs, any agentic harness
(casehub-aml, casehub-clinical, casehub-devtown, QuarkMind, and any future harness).

**Building the CaseHub platform itself?** Read [FOUNDATION-INDEX.md](FOUNDATION-INDEX.md) instead.

Reconstitute this index: `grep -rl "^scope: application" docs/protocols/*.md`

---

## Protocols

| File | Rule | Applies to |
|------|------|------------|
| [platform-module-progression.md](casehub/platform-module-progression.md) | Adopt casehub-platform modules progressively — mock by default, real (oidc/config/jpa/scim) only when the concern is production-ready | any repo consuming casehub-platform |
| [layer-log.md](universal/layer-log.md) | Maintain LAYER-LOG.md as definition of done per harness layer — structured wiring, gotchas, and pattern-to-replicate for each layer | All CaseHub domain applications |
| [case-definition-layers.md](casehub/case-definition-layers.md) | Three-layer case definition architecture — YAML (structure) + `*CaseDescriptor` POJO (business logic); `*CaseDefinitions` FuncDSL companions superseded for new harnesses after casehub-life#27; LambdaExpressionEvaluator only in DSL companions and tests | All CaseHub domain applications defining CasePlanModels |
| [worker-function-execution-model.md](casehub/worker-function-execution-model.md) | Worker functions must use `FuncWorkflowBuilder.workflow().tasks(...).build()` — never raw lambdas in production; choose FuncDSL task type by operation (function/agent/get) | All CaseHub domain applications defining Worker functions |
| [casehub-work-illegal-state-exception.md](casehub/casehub-work-illegal-state-exception.md) | Do not throw IllegalStateException in REST-reachable code — casehub-work maps it to HTTP 409 via IllegalStateExceptionMapper | All harnesses using casehub-work |
| [harness-ledger-writer.md](casehub/harness-ledger-writer.md) | Extract a dedicated `@ApplicationScoped` writer bean that owns `sequenceNumber` computation when more than one service writes entries of the same `LedgerEntry` subtype for the same subject | Harnesses with multi-service ledger writes |
| [qhorus-inbound-normaliser-channel-scope.md](casehub/qhorus-inbound-normaliser-channel-scope.md) | `InboundNormaliser` implementations must scope to relevant channel patterns — check `ChannelRef.name()` before applying domain-specific detection; default unrecognised channels to `MessageType.QUERY` | Any harness implementing `InboundNormaliser` |
| [qhorus-per-entity-governance-channels.md](casehub/qhorus-per-entity-governance-channels.md) | Name governance oversight channels after the entity being governed, not the actor — channel name is the only reliable entity correlator until qhorus#154 ships | Any harness issuing COMMANDs via qhorus for governance decisions |
| [hexagonal-application-service-placement.md](casehub/hexagonal-application-service-placement.md) | Use-case orchestration lives in `app/`, `api/` stays pure domain with zero foundation deps — result types crossing the boundary are primitive-only Java records | Application-tier repos (aml, clinical, devtown) |
| [casehub-work-hibernate-packages.md](casehub/casehub-work-hibernate-packages.md) | casehub-work Hibernate scan requires both `runtime.model` and `runtime.filter` — omitting filter causes silent `FilterRule entity not found` at startup | All harnesses using casehub-work |
| [flyway-consumer-numbering.md](casehub/flyway-consumer-numbering.md) | Apps embedding casehub-work must start domain migrations at V100+ — casehub-work owns V1–V99; no build-time warning exists for collisions | All harnesses embedding casehub-work |
| [multi-datasource-ledger-work-qhorus.md](casehub/multi-datasource-ledger-work-qhorus.md) | Apps using casehub-ledger + casehub-work + casehub-qhorus together require two Hibernate PUs: default (work + domain) and qhorus (qhorus + ledger); route ledger with `casehub.ledger.datasource=qhorus` | Any harness using all three foundation extensions |
| [coordinator-no-transactional-multi-datasource.md](casehub/coordinator-no-transactional-multi-datasource.md) | Do not use @Transactional at the coordinator level when the method spans default + qhorus datasources — per-service transaction boundaries only; XA fails with H2 | Any CaseHub application coordinator spanning multiple persistence units |
| [harness-rest-resource-blocking-applicationscoped.md](casehub/harness-rest-resource-blocking-applicationscoped.md) | REST resource classes using quarkus-rest with JDBC ORM must be @Blocking @ApplicationScoped — @Blocking prevents I/O thread blocking; @ApplicationScoped enables auth-retrofit CDI interceptors | All harnesses using quarkus-rest (RESTEasy Reactive) with JDBC Panache |
| [harness-transactional-service-layer-only.md](casehub/harness-transactional-service-layer-only.md) | @Transactional belongs on service methods only — never on REST resource methods; placing it on the resource hides TOCTOU races and creates hidden cross-boundary coupling | All casehub harness applications with a service + REST resource layer |
| [harness-tenantid-explicit-parameter.md](casehub/harness-tenantid-explicit-parameter.md) | Services callable from Quartz or async CDI observers must receive tenantId as an explicit parameter — never inject @RequestScoped CurrentPrincipal | Any @ApplicationScoped service called from scheduler or async observer paths |
| [secondary-write-isolation-after-terminal-commit.md](casehub/secondary-write-isolation-after-terminal-commit.md) | Secondary audit writes after a terminal-status REQUIRES_NEW must run outside the primary try-catch — violations cause post-commit status regression and duplicate delivery | Any harness service committing entity state with REQUIRES_NEW then writing secondary audit entries |
| [harness-jdbc-only-reactive-suppression.md](casehub/harness-jdbc-only-reactive-suppression.md) | JDBC-only harnesses must explicitly set `quarkus.datasource.reactive=false` (per datasource) in production `application.properties`; without it, reactive beans from transitive deps cause 30+ unsatisfied CDI errors at build time | clinical, aml, devtown running JDBC-only |
| [harness-trust-policy-source-split.md](casehub/harness-trust-policy-source-split.md) | Trust policy base fields (threshold, minimumObservations, borderlineMargin) come from CapabilityRegistry — never duplicated in YAML; blendFactor and qualityFloors (absent from RoutingPolicy) go in YAML | devtown TrustRoutingPolicyProvider; any harness implementing TrustRoutingPolicyProvider |
| [trust-maturity-model.md](casehub/trust-maturity-model.md) | Four-phase trust maturity model — Phase 0 is availability routing (day-1 Gastown parity), phases advance automatically as `minimumObservations` thresholds are crossed; every capability must declare a `fallbackType`; never block on cold-start | Any harness using trust-based worker routing |
| [work-adapter-memory-plan-item-store.md](casehub/work-adapter-memory-plan-item-store.md) | When casehub-engine-work-adapter is on the test classpath, add MemoryPlanItemStore to quarkus.arc.selected-alternatives — omitting it causes HumanTaskScheduleHandler.handleInlineMode() to silently roll back WorkItem creation via JPA throw | devtown @QuarkusTest modules indexing casehub-engine-work-adapter |
| [hitl-test-context-pre-seeding.md](casehub/hitl-test-context-pre-seeding.md) | Pre-seed parallel check keys with non-null values in HITL integration tests — missing values cause capability bindings to fire tryProvision() and block the Vert.x event loop, triggering WorkItemLifecycleAdapter timeout | @QuarkusTest classes exercising humanTask bindings in devtown CasePlanModels |
| [engine-runtime-cdi-assembly.md](casehub/engine-runtime-cdi-assembly.md) | casehub-engine runtime CDI requires %prod.-scoped index-dependency for engine and engine-common, plus casehub-platform-expression as a production dep (engine#316) — neither module ships Jandex | All CaseHub harnesses depending on casehub-engine |
| [engine-worker-event-observer-async.md](casehub/engine-worker-event-observer-async.md) | Harness observers of WorkerDecisionEvent (and any other engine event fired via fireAsync()) must use @ObservesAsync — @Observes is silently never called for async events | Any harness CDI bean observing casehub-engine worker events |
| [ledger-entry-non-merkle-use-entity-manager.md](casehub/ledger-entry-non-merkle-use-entity-manager.md) | LedgerEntry subclasses that should NOT appear in the Merkle audit chain must persist via EntityManager.persist(), not LedgerEntryRepository.save() — save() triggers the Merkle enricher pipeline, causing concurrent constraint violations | Harnesses writing LedgerEntry subclasses for internal records (attestations, routing metadata) |
| [domain-supplement-pattern.md](casehub/domain-supplement-pattern.md) | Attach domain context to foundation primitives via a supplement table — not a wrapper entity; wrapper entities become redundant at Layer 5 | Application-tier repos (life, aml, clinical, devtown) |
| [harness-workitem-template-test-seeding.md](casehub/harness-workitem-template-test-seeding.md) | Seed WorkItemTemplates in @QuarkusTest via @BeforeEach @Transactional — Flyway V-seeds don't run in test mode (drop-and-create) | Any harness @QuarkusTest using WorkItemTemplate-based task creation |
| [domain-event-not-workitem-lifecycle-for-triggers.md](casehub/domain-event-not-workitem-lifecycle-for-triggers.md) | Observe domain CDI events (fire once per domain operation), not WorkItemLifecycleEvent (fires once per WorkItem — N times when engine creates N WorkItems) | Harness notification listeners and side-effect observers using @ObservesAsync |
| [rest-domain-validation-two-mapper-pattern.md](casehub/rest-domain-validation-two-mapper-pattern.md) | Domain validation errors from record compact constructors require two JAX-RS exception mappers in tandem: ExceptionMapper<IllegalArgumentException> + ExceptionMapper<JsonMappingException> | CaseHub application REST layer using record compact constructors for domain validation |
| [journal-section-anchor.md](casehub/journal-section-anchor.md) | Every `design/JOURNAL.md` entry header must carry a `§SectionName` anchor matching a `##` heading in `DESIGN.md` — entries without it are silently skipped at epic close | All CaseHub workspace repos using design/JOURNAL.md and the epic close workflow |
| *(pending)* | Layered adoption approach — one foundation module at a time; each layer independently runnable with a single HTTP call | All CaseHub domain applications |
| *(pending)* | Production-first — do not design or architect for the tutorial; the tutorial documents what you built | All CaseHub domain applications |

---

## Garden References — CaseHub-Specific

Gotchas and techniques discovered while building on CaseHub foundation modules.
For generic Quarkus/Java entries (CDI, testing, migrations), see [FOUNDATION-INDEX.md §Garden](FOUNDATION-INDEX.md).

### casehub-engine

| GE-ID | Title |
|---|---|
| GE-20260414-10d4da | CNCF Serverless Workflow CallableTaskBuilder.accept(Class) cannot distinguish custom callable names |
| GE-20260414-f4f539 | CaseHubReactor.startCase() no longer calls registerCaseDefinition() — definitions only register at startup |
| GE-20260417-3887be | Reset shared test counter immediately after a blocking startCase() call to minimise async contamination |
| GE-20260417-4a3c22 | Worker lambda receives null for context fields added to inputSchema — keys may not survive event log serialization |
| GE-20260417-d67b22 | Use per-case DB query instead of shared AtomicInteger to isolate @QuarkusTest async worker assertions |
| GE-20260420-18fbd4 | ExpressionEvaluator is a marker-only interface — actual evaluation requires instanceof dispatch to LambdaExpressionEvaluator.test() |
| GE-20260420-4a62d3 | casehub-persistence-memory as Maven test dependency fails for @QuarkusTest — copy sources instead |
| GE-20260421-88296e | persistence-memory Maven profile required for all engine tests without Docker |
| GE-20260428-9311f8 | @ApplicationScoped no-op SPI beans collide with consumer implementations when engine is indexed |
| GE-20260428-9571b8 | Bayesian Beta trust model may store confidence as a field but not use it in the update weight |
| GE-20260428-a67806 | Vert.x event-bus handlers lack @Blocking — JPA consumer calls fail from IO thread |
| GE-20260429-a9bd85 | CaseInstanceRepository.updateStateAndAppendEvent() already appends the EventLog — calling append() first duplicates the write |
| GE-20260512-59a501 | CaseContextImpl.snapshot() returns CaseContextImpl — subclasses lose their type on copy |
| GE-20260512-5bcc7b | Preserve subclass type in CaseContextImpl.snapshot() without accessing private deepCopy |
| GE-20260512-b0eea3 | CaseContextImpl.set(key, null) on an absent key is a no-op — the key is never inserted |
| GE-20260531-864d8e | @Observes silently never fires for casehub-engine WorkerDecisionEvent — must use @ObservesAsync |

### casehub-work

| GE-ID | Title |
|---|---|
| GE-20260421-4a9364 | JpaWorkItemStore.scan() with assigneeId also matches candidateUsers LIKE '%actorId%' |
| GE-20260421-9498ff | WorkItemService.delegate() must run strategy BEFORE clearing assigneeId or Hibernate auto-flush corrupts workload counts |
| GE-20260423-3be346 | WorkerCandidate.of(id) creates empty capabilities — WorkBroker filters all candidates when requiredCapabilities is non-null |
| GE-20260427-5d7c67 | quarkus-work (full) brings JpaWorkloadProvider that clashes with any other WorkloadProvider bean |
| GE-20260427-bf4338 | WorkItemStatus.EXPIRED.isTerminal() returns false — EXPIRED is not treated as terminal by quarkus-work |
| GE-20260427-cc77a7 | WorkItemLifecycleEvent.workItem() doesn't exist — access WorkItem via source() cast |
| GE-20260429-cd60ee | Add completeFromSystem()/rejectFromSystem() to WorkItemService to bypass human-actor lifecycle guards |
| GE-20260501-29e3b8 | QuarkusTest: notification rules persist across tests — dynamic WireMock port reuse causes false positives |
| GE-20260502-c77725 | MultiInstanceSpawnService.onThresholdReached defaults to CANCEL — tests completing all children race with coordinator |
| GE-20260511-a28064 | Quarkus Flyway classpath:db/migration scans transitive JARs — casehub-work V1-V21 conflicts with consumer domain migrations |

### casehub-ledger

| GE-ID | Title |
|---|---|
| GE-20260420-b9259e | LedgerAttestation in quarkus-ledger is plain @Entity — Panache statics cause compile error |
| GE-20260424-6b88a0 | quarkus.ledger.datasource routes LedgerEntityManagerProducer to a named PU — not documented |
| GE-20260427-97650e | CDI ambiguity when adding second implementation of a quarkus-ledger repository interface |
| GE-20260429-2e1c4f | quarkus-ledger sequence_number index is not unique — race yields silent duplicate sequences |
| GE-20260531-d2ed26 | LedgerEntryRepository.save() triggers full Merkle chain update — concurrent writes violate UQ_MERKLE_FRONTIER_SUBJECT_LEVEL |
| GE-20260531-1587fe | JpaLedgerMerkleFrontierRepository must be in selected-alternatives alongside JpaLedgerEntryRepository for LedgerVerificationService |
| GE-20260531-46f8ab | casehub.ledger.identity.tokenisation.enabled=true required in tests for LedgerErasureService.erase() to do anything |

### casehub-qhorus

| GE-ID | Title |
|---|---|
| GE-20260414-23982b | check_messages excludes EVENT messages by design — tests expecting EVENTs always get fewer results than sent |
| GE-20260501-11ce7f | MessageLedgerEntry.content is null for EVENT entries — LIKE content search silently returns nothing |
| GE-20260501-b12416 | MessageLedgerEntry.sequenceNumber is per-channel, not global — wrong ORDER BY for cross-channel queries |
| GE-20260508-492336 | casehub-qhorus activates quarkus-hibernate-reactive unconditionally — fails with JDBC H2 at startup |

### AML-specific (Layer 2 — casehub-work wiring)

| GE-ID | Title |
|---|---|
| GE-20260513-74dc72 | FilterRule scan package — casehub-work Hibernate scan requires io.casehub.work.runtime.filter alongside runtime.model |
| GE-20260513-4f26a7 | @DefaultBean layer displacement — Layer N service with @DefaultBean is displaced by Layer N+1 without @DefaultBean |
| [engine-case-start-three-phase-pattern.md](casehub/engine-case-start-three-phase-pattern.md) | Engine case start requires three-phase @Transactional split — never join() inside a transaction | Any harness service that calls CaseHubRuntime.startCase() or YamlCaseHub.startCase() |
| [engine-investigation-test-drain.md](casehub/engine-investigation-test-drain.md) | Every @QuarkusTest that starts an engine investigation must drain to 'completed' before returning | Any casehub application @QuarkusTest starting a case via AmlEngineCoordinator or CaseHubRuntime |
| [ledger-hash-chain-disabled-in-h2-tests.md](casehub/ledger-hash-chain-disabled-in-h2-tests.md) | Disable Merkle hash chain in H2-backed @QuarkusTest suites | Any casehub application @QuarkusTest suite using H2 and casehub-ledger JPA entities |
| [ledger-tokenise-for-query-optional-contract.md](casehub/ledger-tokenise-for-query-optional-contract.md) | ActorIdentityProvider.tokeniseForQuery() returns Optional.empty() for null input only — present means always proceed with query (token or raw actorId) | casehub-ledger and any consumer providing a custom ActorIdentityProvider |
| [qhorus-dispatch-exception-sanitization.md](casehub/qhorus-dispatch-exception-sanitization.md) | Exception messages must never reach MessageService.dispatch() content on error paths | Any application-tier ChannelBackend or service dispatching DECLINE or FAILURE to a Qhorus channel |
| [quarkus-test-lifecycle-entity-setup.md](casehub/quarkus-test-lifecycle-entity-setup.md) | @QuarkusTest lifecycle tests for @ObservesAsync services must persist entities in @BeforeEach @Transactional | @QuarkusTest tests where the service under test calls Panache.findById() in an @ObservesAsync handler |
| [quarkus-test-spi-override-injectmock.md](casehub/quarkus-test-spi-override-injectmock.md) | Use @InjectMock for SPI override tests — not @TestProfile + @Alternative | Any @QuarkusTest that needs to override a CaseHub SPI or @DefaultBean implementation |
| [tutorial-layer-cdi-displacement.md](casehub/tutorial-layer-cdi-displacement.md) | Tutorial harness layers use @DefaultBean (Layer 1) + @Alternative @Priority(N) (Layer N>1) for CDI displacement | devtown, aml, clinical — any CaseHub tutorial harness implementing a layered learning progression |
| [vertical-slice-planning.md](casehub/vertical-slice-planning.md) | Slice-Indexed Architecture Log (SIAL) — vertical slices as architecture log entries | Any CaseHub application; general best practice for any layered Quarkus application |
| [workload-provider-stub-required-in-tests.md](casehub/workload-provider-stub-required-in-tests.md) | Harness @QuarkusTest must supply @DefaultBean WorkloadProvider stub | Any harness application including casehub-engine in @QuarkusTest scope |
| [descriptor-handler-pattern.md](casehub/descriptor-handler-pattern.md) | Enum values with distinct behaviour (routing, SLA, capabilities, worker logic) belong in `*CaseDescriptor` POJOs + optional CDI handlers — never in switch statements scattered across service classes | casehubio application repos (aml, clinical, devtown, life, quarkmind) |
| [oversight-action-gate-dedicated-hub.md](casehub/oversight-action-gate-dedicated-hub.md) | Oversight action gates (ActionRiskClassifier) use a dedicated YamlCaseHub — never programmatic bindings on an existing case definition | Any harness implementing ActionRiskClassifier gates for consequential agent actions |
