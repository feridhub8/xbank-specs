# Proposal

## Why

Register (two changes from now) calls Keycloak with the `xbank-ms-auth` client secret. That secret must never be in a repo or in plain configuration: ms-auth has to read it from Vault at startup with its own AppRole. This change wires that up on its own, before any code uses the secret, so it can be reviewed and tested in isolation.

## What Changes

- ms-auth loads `secret/xbank/<env>/ms-auth` from Vault at startup with Spring Cloud Vault, signing in with AppRole `ms-auth-<env>`. The key `keycloak-client-secret` becomes the setting `xbank.keycloak.client-secret`.
- If Vault cannot be reached or denies access, ms-auth does not start.
- AppRole credentials come from `.env` (local, read by the app itself), `.env.dev`, and `.env.prod` (compose `env_file`). `.env.example` lists the keys with empty values. None of these files are committed.
- The Keycloak client secrets are written to Vault once: `local` and `dev` get the `xbankDev` realm's secret, `prod` gets the `xbank` realm's.
- Existing tests (health, profiles, container) now start a Vault test container, because the app needs Vault to start.
- README: configuration and how to get the AppRole credentials.

Out of scope: any Keycloak call, the error format, any `/api` endpoint.

## Capabilities

### New Capabilities

None.

### Modified Capabilities

- `secrets-store`: services load their secrets from their own path at startup and refuse to start without them.

## Impact

- `xbank-ms-auth`: `build.gradle` (Spring Cloud BOM, Vault starter, Testcontainers Vault module), `application*.yml`, a properties class, compose files (`env_file`), `.env.example`, tests, README.
- Running Vault: three secrets written (`secret/xbank/{local,dev,prod}/ms-auth`).
- Tests read `../xbank-infrastructure/vault/setup/init-auth.sh` to set up the Vault test container exactly like the real one.
