# Spec Delta

## Purpose

Gives every XBank HTTP API one error format, so the frontend can recognize any error by a stable code and show field errors next to the right input.

## ADDED Requirements

### Requirement: ProblemDetail format
Every error response SHALL have content type `application/problem+json` and the fields `type`, `title`, `status`, `detail`, `instance`, and `code`. `code` SHALL be a stable UPPER_SNAKE_CASE identifier, `status` SHALL equal the HTTP status, `instance` SHALL be the request path, and `type` SHALL be `https://xbank.local/errors/<code in kebab-case>`. Texts SHALL be in English.

#### Scenario: All fields present
- **WHEN** `GET /does-not-exist` is called
- **THEN** the response is `404`, `application/problem+json`, with `code` `NOT_FOUND`, `status` `404`, `type` `https://xbank.local/errors/not-found`, `instance` `/does-not-exist`, and a non-empty `title` and `detail`

### Requirement: Field errors
A request whose body fails validation SHALL return `400` with code `VALIDATION_FAILED` and an `errors` array of `{field, message}` objects, one per invalid field.

#### Scenario: Two invalid fields
- **WHEN** a request body has two invalid fields
- **THEN** the response is `400` with code `VALIDATION_FAILED` and `errors` has one entry for each field, naming it

### Requirement: Malformed request body
A request body that cannot be read (for example invalid JSON) SHALL return `400` with code `MALFORMED_REQUEST`.

#### Scenario: Invalid JSON
- **WHEN** a request is sent with the body `{not json`
- **THEN** the response is `400` with code `MALFORMED_REQUEST`

### Requirement: Unknown path and method
A path that does not exist SHALL return `404` with code `NOT_FOUND`. A method that a path does not support SHALL return `405` with code `METHOD_NOT_ALLOWED`.

#### Scenario: Unknown path
- **WHEN** `GET /does-not-exist` is called
- **THEN** the response is `404`, `application/problem+json`, with code `NOT_FOUND`

#### Scenario: Wrong method
- **WHEN** `POST /actuator/health` is called
- **THEN** the response is `405`, `application/problem+json`, with code `METHOD_NOT_ALLOWED`

### Requirement: No internal details leak
An unexpected failure SHALL return `500` with code `INTERNAL_ERROR` and a generic `detail`, without stack traces, exception class names, or exception messages.

#### Scenario: Unexpected exception
- **WHEN** a request fails with an unexpected exception whose message is `secret internal message`
- **THEN** the response is `500` with code `INTERNAL_ERROR` and the body contains neither that message nor the exception class name

### Requirement: Upstream unavailable
When a service the request depends on cannot be reached, the response SHALL be `503` with code `UPSTREAM_UNAVAILABLE`.

#### Scenario: Dependency down
- **WHEN** a request fails because an upstream service cannot be reached
- **THEN** the response is `503` with code `UPSTREAM_UNAVAILABLE`
