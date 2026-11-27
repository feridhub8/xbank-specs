# Proposal

## Why

The frontend developer and the reviewer need to see what ms-auth's API accepts and returns, and try it, without reading the code. The upcoming login, refresh, logout, and `me` endpoints are also easier to test from a browser. API docs must not be public in production.

## What Changes

- OpenAPI docs with springdoc: `GET /v3/api-docs` (JSON) and `GET /swagger-ui.html` (browser UI), in the `local` and `dev` profiles.
- In `prod` both paths return `404` (ProblemDetail, code `NOT_FOUND`).
- `POST /api/v1/auth/register` is documented: summary, request body, and every response (`201`, `400`, `409`, `503`), with error responses shown as ProblemDetail.
- Global rule in `sdd-global-rules-java/java.md`: every HTTP service exposes OpenAPI docs outside production and documents each endpoint's responses, including errors.
- README: where the docs are.

Out of scope: authorizing with a token in the UI (arrives with Spring Security), docs for endpoints that do not exist yet.

## Capabilities

### New Capabilities

- `api-docs`: machine-readable and browsable API documentation, available outside production only.

### Modified Capabilities

None.

## Impact

- `xbank-ms-auth`: `build.gradle` (springdoc), `common/config/v1/OpenApiConfig`, annotations on `AuthController`, `application-prod.yml`, tests, README.
- `sdd-global-rules-java`: one line in `java.md`.
