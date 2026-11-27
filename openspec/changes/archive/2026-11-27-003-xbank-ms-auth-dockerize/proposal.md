# Proposal

## Why

`xbank-ms-auth` runs only from the IDE today. The `dev` and `prod` environments run in Docker on `xbank_network`, next to Keycloak and Vault, so the service needs an image, compose files, and the two profiles before any feature that talks to those tools.

## What Changes

- Profiles `dev` and `prod` (`application-dev.yml`, `application-prod.yml`). Inside the container the service listens on 8080.
- Multi-stage `Dockerfile` (build with the Gradle wrapper, run on a JRE as a non-root user) and `.dockerignore`.
- `docker-compose.dev.yml` (profile `dev`, host port 9101) and `docker-compose.yml` (profile `prod`, host port 9201). Both use the Compose project name `xbank`, set no `container_name`, and join the external network `xbank_network`.
- Automated tests for the profiles, the image, and the compose files.
- README: how to run dev and prod.

Out of scope: Keycloak, Vault, security, any `/api` endpoint, `.env` files (nothing to configure yet).

## Capabilities

### New Capabilities

None.

### Modified Capabilities

- `service-health`: adds the container port, the dev and prod host ports, and how a service runs in Docker.

## Impact

- `xbank-ms-auth`: new `Dockerfile`, `.dockerignore`, two compose files, two profile files, tests, README; `build.gradle` gets the Testcontainers test dependency.
- Requires Docker and the network `xbank_network` to run dev and prod.
