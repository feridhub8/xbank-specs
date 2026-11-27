# Design

## Context

See proposal.md. `xbank-ms-auth` has one commit: Spring Boot 4.2.0, webmvc and actuator, profile `local` on 9001, tests `HealthIT` and `LocalProfileTest`. Docker 29 runs locally and `xbank_network` exists. Ports 9101 and 9201 are free.

## Goals / Non-Goals

**Goals:**
- `docker compose -f docker-compose.dev.yml up -d --build` and `docker compose up -d --build` start the service with nothing else installed.
- Every requirement checked by `./gradlew test`.

**Non-Goals:**
- `.env` files, Keycloak, Vault: nothing to configure yet. They arrive with the first feature that needs them.
- Image publishing to a registry.

## Decisions

### Profiles
`application-dev.yml` and `application-prod.yml` set `server.port: 8080` explicitly, so the container port is visible in the profile even though it is Spring's default. Later changes add Keycloak and Vault URLs here.

### Dockerfile
Multi-stage:
1. `eclipse-temurin:25-jdk`: copy the wrapper and build files first, download dependencies (cached layer), then copy `src` and run `./gradlew bootJar`.
2. `eclipse-temurin:25-jre`: copy the jar, create user `app`, run as `app`, `EXPOSE 8080`.
The profile is chosen at runtime with `SPRING_PROFILES_ACTIVE`, so one image serves dev and prod.

Alternative considered: copy a jar built on the host. Rejected: compose would then depend on running `./gradlew bootJar` first and on a local JDK.

### Compose files
One service per file, named after the environment (`ms-auth-dev`, `ms-auth-prod`) so both can run at once in the same `xbank` project. `build: .`, `SPRING_PROFILES_ACTIVE`, the port mapping, `restart: unless-stopped`, and `xbank_network` (external).

### Tests
- `ProfilesTest` (replaces `LocalProfileTest`): loads each profile's configuration without a server and checks `server.port` (local 9001, dev 8080, prod 8080).
- `ContainerIT`: builds the image from the `Dockerfile` with Testcontainers (`ImageFromDockerfile`), runs it once with `dev` and once with `prod`, checks health on the mapped container port 8080, and checks the process user is not root. Needs Docker; the build makes it the slowest test (about a minute on a cold cache).
- `ComposeFileTest`: parses both compose files with SnakeYAML (already on the test classpath through Spring Boot) and checks profile, port mapping, `name: xbank`, no `container_name`, and the external network. Running compose itself in a test would clash with an instance already running on 9101/9201.

New test dependency: `org.testcontainers:testcontainers-junit-jupiter` (version from the Spring Boot BOM).

## Risks / Trade-offs

- [ContainerIT is slow because it builds the image] → Accepted; it is the only test that proves the Dockerfile works. Docker layer caching makes later runs faster.
- [ComposeFileTest checks the files, not a running compose stack] → The image itself is tested by ContainerIT; the compose files only add ports, profile, and network, which the test checks directly.
