# Design

## Context

See proposal.md. `xbank-ms-auth` has webmvc, actuator, validation, Lombok, Spring Cloud Vault, and `common/config/v1/KeycloakProperties`. There are no controllers yet; today an unknown path returns Spring's default error JSON (`{"timestamp":...,"status":404,...}`).

## Goals / Non-Goals

**Goals:**
- One place that turns every exception into the same ProblemDetail shape.
- Features add errors by adding a code and an exception class, nothing else.

**Non-Goals:**
- `401`/`403`: need Spring Security, which arrives with the first protected endpoint; that change routes them through the same handler.
- Localized messages: the frontend maps `code` to its own texts.

## Decisions

### Files (`common/exception/v1`)
- `ErrorCode`: enum of `HttpStatus` + English `title`. Starts with the six general codes; features add theirs.
- `ApiException`: base `RuntimeException` with an `ErrorCode`, a `detail` message, and an optional list of `FieldViolation`.
- `FieldViolation`: `field` + `message` (regular class with Lombok, per the global rule on DTOs).
- `UpstreamUnavailableException`: `ApiException` with `UPSTREAM_UNAVAILABLE`, used by clients when a dependency is unreachable.
- `GlobalExceptionHandler`: `@RestControllerAdvice` with one private `build(code, detail, violations, request)` method that every handler uses.

### Handlers
| Exception | Code |
|---|---|
| `ApiException` | its own |
| `MethodArgumentNotValidException` | `VALIDATION_FAILED`, `errors` from the field errors |
| `HttpMessageNotReadableException` | `MALFORMED_REQUEST` |
| `NoResourceFoundException` | `NOT_FOUND` |
| `HttpRequestMethodNotSupportedException` | `METHOD_NOT_ALLOWED` |
| any other `Exception` | `INTERNAL_ERROR`, logged with the stack trace, generic detail |

Spring's built-in `ProblemDetail` class is used; `code` and `errors` are added as properties, so they appear as top-level JSON fields.

Alternative considered: extending `ResponseEntityExceptionHandler`. Rejected: it builds its own titles and details for many exceptions, and overriding each to add `code` is more code than handling the few we need.

### Tests
- `GlobalExceptionHandlerTest`: standalone MockMvc with a test-only controller that throws each case the app cannot produce yet without endpoints: validation with two fields, invalid JSON, unexpected exception, upstream unreachable. Fast, no Vault needed.
- `ErrorsIT`: the real application (`@SpringBootTest`, `TestVault`) for the cases that exist without any controller: unknown path `404` (checking every field: `type`, `title`, `status`, `detail`, `instance`, `code`) and `POST /actuator/health` `405`, both `application/problem+json`.

## Risks / Trade-offs

- [Actuator errors go through the same handler] → Wanted: one format everywhere. `ErrorsIT` checks it.
- [Field errors, 500, and 503 are tested through a test controller, not a real endpoint] → The handler is the same; register's integration tests add the real-endpoint checks.
