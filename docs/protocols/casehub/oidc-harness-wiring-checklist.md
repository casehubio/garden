---
id: PP-20260622-eb234d
title: "OIDC harness wiring checklist — four mandatory steps when adding casehub-platform-oidc, plus SecurityIdentity attribute convention for non-OIDC mechanisms"
type: rule
scope: application
applies_to: "All casehub harness applications (life, devtown, clinical, openclaw) adding casehub-platform-oidc"
severity: important
refs:
  - casehub/auth-retrofit-readiness.md
violation_hint: "All endpoints returning 401 in tests (missing @TestSecurity); all endpoints returning 401 in dev mode (missing dev-profile auth bypass); MissingTenancyException from non-OIDC mechanisms (missing SecurityIdentity attributes)"
created: 2026-06-22
updated: 2026-06-30
---

When adding `casehub-platform-oidc` as a compile dep to a harness application, four steps are mandatory: (1) add `quarkus.oidc.application-type=service` and document the required deployment env vars (`QUARKUS_OIDC_AUTH_SERVER_URL`, `QUARKUS_OIDC_CLIENT_ID`) without hardcoding values; (2) add dev-profile bypass — `%dev.quarkus.security.auth.enabled-in-dev-mode=false` and `%dev.quarkus.oidc.enabled=false` — so endpoints are accessible in dev without a real OIDC server (see GE-20260622-580d45); (3) add discovery-disabled OIDC test config (`quarkus.oidc.discovery-enabled=false`, `quarkus.oidc.jwks-path=protocol/openid-connect/certs`, `quarkus.keycloak.devservices.enabled=false`) to the test `application.properties`; (4) add class-level `@TestSecurity(user=..., roles={...})` to ALL `@QuarkusTest` classes that make HTTP calls to `@RolesAllowed`-guarded endpoints — missing any one class causes intermittent 401 failures. Apply all four steps atomically; partial adoption leaves the application in a broken state.

**Removed in platform#112:** Step 1 (exclude `TenantScopedPrincipal` / `QhorusInboundCurrentPrincipal` via `quarkus.arc.exclude-types`) is no longer needed. Since platform#111, `OidcCurrentPrincipal` is `@Alternative @Priority(100)` and automatically displaces all non-alternative `CurrentPrincipal` implementations. Remove any `exclude-types` entries for `TenantScopedPrincipal`, `QhorusInboundCurrentPrincipal`, `MockCurrentPrincipal`, or `DefaultTestPrincipal` from `application.properties` when `casehub-platform-oidc` is on the classpath.

**Renamed in platform#121:** `OidcCurrentPrincipal` → `SecurityIdentityCurrentPrincipal`. The class no longer requires a JWT principal — it resolves `tenancyId` and `crossTenantAdmin` via a three-tier fallback: (1) anonymous → sentinel values, (2) JWT claim (authoritative when the principal is a `JsonWebToken`), (3) SecurityIdentity attribute (fallback for non-OIDC mechanisms), (4) neither → `MissingTenancyException` / `false`. The module name (`casehub-platform-oidc`) is unchanged.

**SecurityIdentity attribute convention (platform#121):** Non-OIDC `HttpAuthenticationMechanism` implementations and `SecurityIdentityAugmentor` beans stamp tenancy via `QuarkusSecurityIdentity.Builder.addAttribute()` using the reserved keys in `SecurityIdentityAttributes` (platform-api): `tenancyId` (non-blank `String`, required) and `crossTenantAdmin` (`Boolean`, defaults `false` when absent). Wrong-type attributes throw `IllegalStateException` at read time. Blank or empty `tenancyId` values fall through to the next resolution tier.
