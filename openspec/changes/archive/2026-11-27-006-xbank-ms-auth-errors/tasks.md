# Tasks

## 1. Format

- [x] 1.1 Add `ErrorCode` (six general codes), `FieldViolation`, `ApiException`, and `GlobalExceptionHandler` with the shared `build` method and the `ApiException` and `NoResourceFoundException` handlers; add `ErrorsIT` for the unknown-path scenario with every field; verify `./gradlew test` passes

## 2. Request errors

- [x] 2.1 Add the validation (`VALIDATION_FAILED` with `errors`) and unreadable-body (`MALFORMED_REQUEST`) handlers and `GlobalExceptionHandlerTest` cases for two invalid fields and invalid JSON; verify `./gradlew test` passes
- [x] 2.2 Add the method-not-supported handler (`METHOD_NOT_ALLOWED`) and the `ErrorsIT` case for `POST /actuator/health`; verify `./gradlew test` passes

## 3. Server errors

- [x] 3.1 Add the catch-all handler (`INTERNAL_ERROR`, logged, generic detail) and `UpstreamUnavailableException` (`UPSTREAM_UNAVAILABLE`), with `GlobalExceptionHandlerTest` cases checking that the message and class name do not leak and that upstream gives `503`; verify `./gradlew test` passes

## 4. Docs

- [x] 4.1 Add an "Errors" section to `README.md` (fields, general codes, `errors` for validation); verify the listed codes match `ErrorCode`

## Workflow follow-up

- Stop for review of `xbank-ms-auth`; commit and push after approval.
- Archive this change, then commit `xbank-specs`.
