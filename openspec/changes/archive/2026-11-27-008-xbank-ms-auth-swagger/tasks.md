# Tasks

## 1. Docs on and off

- [x] 1.1 Add springdoc 3.2.1 and `OpenApiConfig`; add `ApiDocsIT` for the JSON and UI in `local`; verify `./gradlew test` passes
- [x] 1.2 Turn both off in `application-prod.yml`; add `ApiDocsProdIT`; verify it fails before the change and passes after

## 2. Register documented

- [x] 2.1 Add `@Operation` and `@ApiResponse` (201, 400, 409, 503) to `register`; add the `ApiDocsIT` case checking summary, codes, and `application/problem+json`; verify `./gradlew test` passes

## 3. Rules and docs

- [x] 3.1 Add the OpenAPI rule to `sdd-global-rules-java/java.md` and a "API docs" note to the README; verify the README paths match the ones `ApiDocsIT` calls

## Workflow follow-up

- Stop for review of `xbank-ms-auth` (and the `java.md` line); commit and push after approval.
- Archive this change, then commit `xbank-specs` and `sdd-global-rules-java`.
