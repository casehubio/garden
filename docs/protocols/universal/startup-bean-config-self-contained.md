---
id: PP-20260615-dbc195
title: "@Startup beans must read config via @ConfigProperty directly — no cross-bean @PostConstruct dependencies"
type: rule
scope: universal
applies_to: "Any @Startup @ApplicationScoped bean that depends on shared configuration or global state"
severity: important
garden_ref: "GE-20260615-d065bf"
created: 2026-06-15
---

CDI provides no initialization ordering guarantee between two `@Startup @ApplicationScoped` beans. A bean that depends on another bean's `@PostConstruct` side effect (e.g. a global parser set via `Path.setDefaultParser()`) will fail intermittently depending on which bean CDI initialises first. Every `@Startup` bean must be self-contained: inject the relevant `@ConfigProperty` fields directly and construct any helpers from those values inside its own `@PostConstruct`. Never read global state that a peer `@Startup` bean is responsible for setting. See GE-20260615-d065bf for the root cause and a before/after example.
