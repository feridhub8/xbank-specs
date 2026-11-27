# Proposal

## Why

`xbank-ms-auth` is the first XBank service. Before any feature, it needs an empty, reviewable skeleton that builds, starts, and reports that it is alive. Every later step (Docker, then one endpoint per change) builds on it, and later services copy it.

## What Changes

- New repo `xbank-ms-auth` with a Gradle (Groovy DSL) build, Java 25, Spring Boot 4.2, and the committed Gradle wrapper.
- Main class `com.xbank.ms.auth.AuthApplication` and an empty package layout following `config.yaml` (no domain code yet).
- Profile `local`: the service listens on host port 9001.
- Health endpoint `GET /actuator/health`.
- One smoke test that starts the application and checks the health endpoint.
- `.gitignore`, Graphify setup, and a short README.

Out of scope: Docker and the `dev` and `prod` profiles (next change, dockerize: they run only in Docker, so their profile files and compose files arrive together), security, Keycloak, Vault, any `/api` endpoint.

## Capabilities

### New Capabilities

- `service-health`: How an XBank service reports that it is running, and on which port it listens per environment.

### Modified Capabilities

None.

## Impact

- New repo: `xbank-ms-auth` (local `XBank/xbank-ms-auth`, GitHub `feridhub8/xbank-ms-auth`).
- No change to `xbank-infrastructure`.
