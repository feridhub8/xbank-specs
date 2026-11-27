# Design

## Context

See proposal.md. There is no `xbank-ms-auth` folder yet. Java 25 and Docker are installed; Gradle is not installed globally, but Gradle 9.9.2 is in the local Gradle cache and can generate the wrapper.

## Goals / Non-Goals

**Goals:**
- The smallest project a reviewer can read in a few minutes: build file, main class, two config files, one test.

**Non-Goals:**
- Docker, security, Keycloak, Vault, domain packages. Each arrives in its own later change.

## Decisions

### Versions
Spring Boot 4.2.0 (latest stable), Gradle 9.9.2 wrapper, Java 25 toolchain. Spring Cloud is not added yet; it comes with Vault.

### Dependencies
Only `spring-boot-starter-webmvc` and `spring-boot-starter-actuator`, plus `spring-boot-starter-webmvc-test` for the test. Lombok, MapStruct, validation, and security are added by the change that first needs them.

### Files
```
xbank-ms-auth/
+-- gradle/wrapper/          (wrapper jar + properties)
+-- gradlew, gradlew.bat
+-- settings.gradle, build.gradle
+-- .gitignore, README.md
+-- src/main/java/com/xbank/ms/auth/AuthApplication.java
+-- src/main/resources/application.yml          (name, actuator: health only)
+-- src/main/resources/application-local.yml    (server.port 9001)
+-- src/test/java/com/xbank/ms/auth/HealthIT.java
```
No `common/` or domain folders yet: empty packages are not committed; they appear with the first code that needs them.

### Test
- `HealthIT`: `@SpringBootTest(webEnvironment = RANDOM_PORT)` with profile `local`, calls `/actuator/health` (expects `UP`) and `/actuator/env` (expects `404`). A random port avoids clashing with a running local instance.
- `LocalProfileTest`: loads the `local` profile's configuration without starting a server and asserts `server.port` is `9001`.

## Risks / Trade-offs

- [The port test checks configuration, not a live socket on 9001] → Spring Boot binds to `server.port`; testing the real socket would clash with a running local instance.
