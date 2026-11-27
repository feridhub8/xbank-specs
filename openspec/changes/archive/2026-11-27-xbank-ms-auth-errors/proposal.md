# Proposal

## Why

The frontend needs to recognize every error from XBank's API the same way: by a stable code, not by parsing messages, and with field errors it can show next to the right input. Register (next change) is the first endpoint that returns errors, so the format must exist first. Every later service copies it.

## What Changes

- Every error response from ms-auth is a ProblemDetail (RFC 9457, `application/problem+json`) with an extra `code` field, and an `errors` list for validation failures. Texts are in English.
- General error codes: `VALIDATION_FAILED`, `MALFORMED_REQUEST`, `NOT_FOUND`, `METHOD_NOT_ALLOWED`, `UPSTREAM_UNAVAILABLE`, `INTERNAL_ERROR`.
- A base exception that features extend with their own codes (for example register adds `FIN_ALREADY_EXISTS`).
- Unexpected errors return a generic message; stack traces, class names, and upstream responses never reach the client. They are logged.
- README: the error format.

Out of scope: `401`/`403` (they arrive with the first protected endpoint, together with Spring Security), feature-specific codes, any `/api` endpoint.

## Capabilities

### New Capabilities

- `api-errors`: the error response format every XBank HTTP API returns.

### Modified Capabilities

None.

## Impact

- `xbank-ms-auth`: new `common/exception/v1` package and tests; README.
- No change to configuration, Docker, or infrastructure.
