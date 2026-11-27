# Tasks

## 1. Test container

- [x] 1.1 Add `tests/` (`docker-compose.yml`, `Dockerfile` with pinned alpine + curl + jq, `run.sh` that runs every `*.sh` test file and fails on any failure) with one smoke test that reaches Keycloak and Vault; verify `docker compose -f tests/docker-compose.yml run --rm tests` passes

## 2. FIN username

- [x] 2.1 Add `tests/keycloak.sh` (duplicate FIN `409`, stored username is the FIN, email login) and run it against the current realms; verify the duplicate-FIN and username tests fail before the realm change
- [x] 2.2 Set `registrationEmailAsUsername: false` in both realm JSONs and re-import both running realms; verify `tests/keycloak.sh` passes

## 3. Vault auth

- [x] 3.1 Add `vault/setup/init-auth.sh`, mount `./setup` in `vault/docker-compose.yml`, add `VAULT_ADMIN_PASSWORD` to `vault/.env.example` (empty) and `vault/.env` (`admin`); recreate the Vault container, unseal, and run the script with the root token; verify it finishes without errors
- [x] 3.2 Add `tests/vault.sh` (AppRole own path not denied, other env `403`, admin userpass login can list policies and write a secret, setup rerun keeps the role ID); verify it passes

## 4. Root token

- [x] 4.1 Add the "no root token" test to `tests/vault.sh`, revoke the root token after a successful `admin` login; verify the full suite passes

## 5. Docs

- [x] 5.1 Update `README.md`: FIN username, Vault setup command, daily login with `admin`, generating a root token from the unseal key, running the tests; verify the commands match the file names and service names in the repo

## Workflow follow-up

- Stop for review of `xbank-infrastructure`; commit and push after approval.
- Commit and push the global rule change in `sdd-global-rules-java` (tests folder allowed in the infrastructure repo) after approval.
- Archive this change, then commit `xbank-specs`.
