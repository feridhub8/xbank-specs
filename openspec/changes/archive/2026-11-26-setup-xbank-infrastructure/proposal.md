# Proposal

## Why

The first XBank feature is user registration and login through `xbank-ms-auth`, backed by Keycloak and with secrets in Vault. None of these tools exist yet, so the infrastructure must come first, in its own repo, before any service is written.

## What Changes

- New GitHub repo `xbank-infrastructure`, holding only the Docker setup of infrastructure tools.
- PostgreSQL as Keycloak's database.
- One Keycloak instance with two realms: `xbank` (prod) and `xbankDev` (local and dev). Each realm has the roles `USER` (default for new users) and `ADMIN` (assigned by hand), a password policy, and a confidential client for `xbank-ms-auth`.
- Realm configuration is committed and imported automatically, so every machine gets the same realms.
- HashiCorp Vault with persistent file storage and a KV v2 engine at `secret/`, using the path layout `secret/xbank/<env>/<service>`.
- A root `docker-compose.yml` that starts and stops every tool together.
- Project-wide decisions (environments, realm naming, secrets layout, infrastructure tools) recorded in `xbank-specs/openspec/config.yaml`.

## Capabilities

### New Capabilities

- `identity-realms`: Keycloak realms per environment, roles, password policy, and the client that backend services use.
- `secrets-store`: Where and how XBank services read their secrets, per environment.

### Modified Capabilities

None.

## Impact

- New repo: `xbank-infrastructure` (local `XBank/xbank-infrastructure`, GitHub `feridhub8/xbank-infrastructure`).
- `xbank-specs/openspec/config.yaml`: the infrastructure items move from "not decided yet" to decided.
- Unblocks the next change, `xbank-ms-auth`, which depends on both realms and on Vault.
- Requires Docker on every machine that runs XBank.
