# Proposal

## Why

The next ms-auth change (register) needs two things the infrastructure does not have yet: Keycloak must use FIN as the username, so it rejects a second account with the same FIN by itself, and ms-auth needs its own Vault login to read the Keycloak client secret. People should also stop using the all-powerful root token for daily work.

## What Changes

- **BREAKING**: both realms use FIN as the username instead of the email (`registrationEmailAsUsername: false`). Email stays unique and is still used to log in. Existing local users must be recreated.
- New `vault/setup/init-auth.sh`, run inside the Vault container: enables AppRole and userpass, creates one AppRole per service and environment (`ms-auth-local`, `ms-auth-dev`, `ms-auth-prod`) that can read only its own path, and the `admin` user with an admin policy. Safe to run again.
- `vault/.env` gains `VAULT_ADMIN_PASSWORD` (empty in `.env.example`).
- The setup is applied to the running Keycloak and Vault, and the root token is revoked.
- New `tests/` folder: a test container started with docker compose that checks the realm and Vault behavior against the running infrastructure.
- README: FIN username, Vault setup, daily login with `admin`, how to run the tests.

Out of scope: ms-auth code, writing the Keycloak client secret into Vault (the register change does that when it first needs it), TLS, Vault auto-unseal.

## Capabilities

### New Capabilities

None.

### Modified Capabilities

- `identity-realms`: users are identified by FIN as their username.
- `secrets-store`: services sign in with their own AppRole, people with userpass, and no root token stays valid.

## Impact

- `xbank-infrastructure`: realm JSONs, `vault/docker-compose.yml` (mounts `setup/`), `vault/.env.example`, new `vault/setup/` and `tests/`, README.
- Running Keycloak: realms re-imported (test users lost). Running Vault: new auth methods, policies, roles, `admin` user; root token revoked.
- Global rules: `microservices.md` already allows a `tests/` folder in the infrastructure repo.
