---
id: PP-20260601-37179a
title: "Every casehub library JAR that ships CDI beans or SPI supertypes must include a Jandex index"
type: rule
scope: platform
applies_to: "all casehub library modules that define @ApplicationScoped, @DefaultBean, or any CDI-annotated class — AND pure-interface/record JARs whose types are implemented by CDI beans in consuming modules"
severity: important
refs:
  - jvm/GE-20260601-7a3b38.md
violation_hint: "module pom.xml has no jandex-maven-plugin; JAR ships without META-INF/jandex.idx"
created: 2026-06-01
---

Any casehub module whose JAR will be consumed as a library dependency must configure the `jandex-maven-plugin` so that `META-INF/jandex.idx` is included in the published artifact. Without an index, Quarkus ARC cannot resolve the CDI type hierarchy for beans defined in that JAR — even if the implementing bean's own JAR is correctly indexed. The failure mode is silent: the bean is simply not wired, and the error ("Unsatisfied dependency") points at the injection site rather than the missing index. This applies not only to JARs that ship CDI beans, but also to pure-SPI JARs that ship only interfaces and records: if an implementing bean in a consuming module declares a supertype from the SPI JAR, ARC must be able to find that supertype in some index — and the SPI JAR is the only place it lives. `casehub-platform-api` is the canonical example. The parent pom declares the plugin version in `<pluginManagement>`; each library module must activate it explicitly in its own `<build>` section.
