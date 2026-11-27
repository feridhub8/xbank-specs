# Design

## Context

See proposal.md. `xbank-infrastructure` has one commit: PostgreSQL 17.12, Keycloak 26.8.2 (realms `xbank`, `xbankDev`, email as username), Vault 1.21.4 (file storage, KV v2 at `secret/`, only the root token exists). Containers are running and Vault is unsealed. The unseal key and root token are in `XBank/vault-init-local.txt`, outside every repo.

## Goals / Non-Goals

**Goals:**
- Realms with FIN as username, reproducible from the committed JSON.
- A Vault auth setup anyone can rerun with one docker compose command.
- Tests that check both against the running infrastructure, started with docker compose.

**Non-Goals:**
- Client secrets in Vault, ms-auth code, TLS, auto-unseal.

## Decisions

### Realms
Set `registrationEmailAsUsername` to `false` in both realm JSONs; nothing else changes. Keycloak already enforces unique usernames and emails (`duplicateEmailsAllowed: false`) and allows email login (`loginWithEmailAllowed: true`). Keycloak stores usernames in lowercase; callers that show the FIN uppercase it (ms-auth's concern). The running realms are deleted and re-imported, because import skips existing realms.

### Vault setup script
`vault/setup/init-auth.sh`, mounted read-only at `/vault/setup`, run with:
```
docker compose exec -e VAULT_TOKEN=<token> vault /vault/setup/init-auth.sh
```
It reads `VAULT_ADMIN_PASSWORD` from the container environment (`vault/.env`). Steps, each skipped when already done:
1. Enable `approle` and `userpass` if missing; enable KV v2 at `secret/` if missing.
2. For each service in `SERVICES="ms-auth"` and env in `local dev prod`: policy `<service>-<env>` with `read` on `secret/data/xbank/<env>/<service>`; AppRole `<service>-<env>` with that policy, token TTL 1h, max 24h. Writing the role again keeps its role ID.
3. Policy `admin` (all paths, all capabilities including `sudo`) and userpass user `admin`.
Adding a service later means adding it to `SERVICES` and rerunning.

### Root token
After the setup and a successful `admin` login, the root token is revoked (`vault token revoke -self`). The unseal key can generate a new one (`vault operator generate-root`) when needed; README documents it.

### Tests
```
tests/
+-- docker-compose.yml   name: xbank, service "tests", build ./, env_file ../keycloak/.env and ../vault/.env, xbank_network
+-- Dockerfile           alpine (pinned) + curl + jq
+-- run.sh               runs every test file, prints PASS/FAIL per test, exits non-zero on any failure
+-- keycloak.sh          FIN-is-username scenarios
+-- vault.sh             AppRole, userpass, repeatable setup, no root token
```
Run with `docker compose -f tests/docker-compose.yml run --rm tests`. Tests talk to `keycloak:8080` and `vault:8200` over `xbank_network`.
- Keycloak tests sign in as the Keycloak admin and create users with a random FIN in `xbankDev`. They check `409` for a duplicate FIN, that the stored username is the FIN, and email login through the realm's existing `xbank-ms-auth` client (its secret read through the admin API, never printed). Every test user is deleted at the end, also on failure (`trap`).
- Vault tests sign in as `admin` with userpass, create a secret ID for `ms-auth-local`, log in with the AppRole, and check `403` on `secret/data/xbank/prod/ms-auth` and not `403` on its own path. "No root token" lists token accessors (`auth/token/accessors`, needs `sudo`) and looks each up, failing if any has the `root` policy. "Repeatable" reads the role ID, reruns the setup with the admin token, and compares.
- The test secret IDs are destroyed at the end.

Alternative considered: Testcontainers in a Java module. Rejected: the repo holds Docker setup only; shell tests in a container keep it that way.

## Risks / Trade-offs

- [Tests run against the live dev realm and Vault, not isolated copies] → They only create and delete their own users and secret IDs, with random names, and clean up on exit.
- [Re-importing realms deletes existing users] → Only test users exist today.
- [Root token revoked; a lost unseal key now also means no way to get admin access back] → The unseal key was already the critical secret; README says to keep it in a password manager.
- [Admin policy is broad] → Test environment; a narrower admin policy is a later decision.

## Migration Plan

1. Apply: delete and re-import both realms, recreate the Vault container (new mount), unseal, run the setup with the root token, log in as `admin`, run the tests, revoke the root token, run the tests again. Commit after review.
2. Rollback: revert the realm JSONs and re-import; disable `approle` and `userpass` with a root token generated from the unseal key.
