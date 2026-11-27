# Design

## Context

See proposal.md. `xbank-ms-auth` (Spring Boot 4.2.0, webmvc, no Spring Security) has one endpoint, `POST /api/v1/auth/register`, and a `GlobalExceptionHandler` that turns unknown paths into `404 NOT_FOUND`. springdoc-openapi 3.x targets Spring Boot 4; the newest release is 3.2.1.

## Goals / Non-Goals

**Goals:**
- Docs on in `local` and `dev`, off in `prod`, decided by profile configuration only.
- Error responses visible in the docs, so the frontend sees the ProblemDetail shape.

**Non-Goals:**
- Bearer-token "Authorize" in the UI (with Spring Security later).
- Generating clients from the spec.

## Decisions

### Dependency
`org.springdoc:springdoc-openapi-starter-webmvc-ui:3.2.1`. If it does not start with Spring Boot 4.2.0, use the newest 3.x that does and record it here.

### On and off
Enabled by default (springdoc's default). `application-prod.yml` sets `springdoc.api-docs.enabled: false` and `springdoc.swagger-ui.enabled: false`; the paths then have no handler, so `GlobalExceptionHandler` answers `404 NOT_FOUND` like any unknown path.

### Description
- `common/config/v1/OpenApiConfig`: an `OpenAPI` bean with title `XBank Auth API`, the app version, and a short description.
- `AuthController.register`: `@Operation(summary = ...)` and `@ApiResponse` for `201` (`UserResponse`), `400`, `409`, `503` (`ProblemDetail`, `application/problem+json`).

### Tests
- `ApiDocsIT` (`local`, `TestVault`): `/v3/api-docs` is `200` with `openapi` starting `3.`; `/swagger-ui.html` followed to `200` HTML; the register path has a summary and responses `201`, `400`, `409`, `503`, the error ones with `application/problem+json`.
- `ApiDocsProdIT` (`prod`, `TestVault` with AppRole `ms-auth-prod`): both paths `404 NOT_FOUND`. The AppRole is set through system properties: `@DynamicPropertySource` values do not exist yet when the Vault config import logs in (found during apply, the app got `403` on the prod path).

springdoc 3.2.1 works with Spring Boot 4.2.0 (checked during apply).

## Risks / Trade-offs

- [Annotations can drift from real behavior] → `RegisterIT` checks real responses; `ApiDocsIT` checks the documented list. A new status needs both.
- [Docs in `dev` are reachable by anyone who reaches port 9101] → Same machine only for now.
