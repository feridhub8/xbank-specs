# api-docs Specification

## Purpose

Gives developers machine-readable and browsable documentation of each service's HTTP API outside production, so they can see and try every endpoint without reading code.

## Requirements

### Requirement: Docs outside production
In the `local` and `dev` profiles a service SHALL serve its OpenAPI description at `GET /v3/api-docs` and a browser UI at `GET /swagger-ui.html`, without authentication.

#### Scenario: OpenAPI JSON in local
- **WHEN** `GET /v3/api-docs` is called on a service running with profile `local`
- **THEN** the response is `200` with an OpenAPI 3 document

#### Scenario: Browser UI in local
- **WHEN** `GET /swagger-ui.html` is called on a service running with profile `local` and redirects are followed
- **THEN** the response is `200` with the Swagger UI page

### Requirement: No docs in production
In the `prod` profile both `/v3/api-docs` and `/swagger-ui.html` SHALL return `404` with code `NOT_FOUND`.

#### Scenario: Docs hidden in prod
- **WHEN** `GET /v3/api-docs` or `GET /swagger-ui.html` is called on a service running with profile `prod`
- **THEN** the response is `404` with code `NOT_FOUND`

### Requirement: Endpoints document their responses
Every endpoint SHALL appear in the OpenAPI description with a summary and every response it can return. Error responses SHALL use content type `application/problem+json`.

#### Scenario: Register is documented
- **WHEN** the OpenAPI description of `xbank-ms-auth` is read
- **THEN** `POST /api/v1/auth/register` has a summary and responses `201`, `400`, `409`, and `503`, and `400`, `409`, and `503` use `application/problem+json`
