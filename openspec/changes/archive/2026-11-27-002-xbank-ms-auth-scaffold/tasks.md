# Tasks

## 1. Skeleton

- [x] 1.1 Create `XBank/xbank-ms-auth`, `git init`, complete `.gitignore`, Gradle 9.9.2 wrapper, `settings.gradle`, `build.gradle` (Spring Boot 4.2.0, Java 25, webmvc, actuator); verify `./gradlew build -x test` succeeds
- [x] 1.2 Add `AuthApplication`, `application.yml` (health only), `application-local.yml` (port 9001), `HealthIT`, and `LocalProfileTest`; verify `./gradlew test` passes. Only the `local` profile here: `dev` and `prod` come with the dockerize change.
- [x] 1.3 Set up Graphify per the global rules and write a short `README.md` (build, run, test); verify `graphify hook status` and that no committed file contains an absolute path

## Workflow follow-up

- Stop for review of `xbank-ms-auth`; commit after approval and create the GitHub repo `feridhub8/xbank-ms-auth`.
- Archive this change, then commit `xbank-specs` (including the pending `config.yaml` decisions).
