# Tasks

## 1. Profiles

- [x] 1.1 Add `application-dev.yml` and `application-prod.yml` (`server.port: 8080`), replace `LocalProfileTest` with `ProfilesTest` covering local 9001, dev 8080, prod 8080; verify `./gradlew test` passes

## 2. Image

- [x] 2.1 Add the multi-stage `Dockerfile` and `.dockerignore`, add `testcontainers-junit-jupiter` to `build.gradle`, add `ContainerIT` (dev and prod health on 8080, non-root user); verify `./gradlew test` passes with Docker running

## 3. Compose

- [x] 3.1 Add `docker-compose.dev.yml` (dev, 9101) and `docker-compose.yml` (prod, 9201) and `ComposeFileTest`; verify `./gradlew test` passes

## 4. Docs

- [x] 4.1 Update `README.md` with the environments table (local 9001, dev 9101, prod 9201), the compose commands, and that `xbank_network` must exist; verify the README commands match the compose file names and ports checked by `ComposeFileTest`

## Workflow follow-up

- Stop for review of `xbank-ms-auth`; commit after approval and push.
- Archive this change, then commit `xbank-specs`.
