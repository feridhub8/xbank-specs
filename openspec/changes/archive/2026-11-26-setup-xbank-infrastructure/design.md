# Design

## Context

See proposal.md for motivation. There is no code yet; `xbank-infrastructure` is a new repo next to `xbank-specs` in the local `XBank/` folder. Only Docker is assumed on the host.

## Goals / Non-Goals

**Goals:**
- One `docker compose up -d` in the repo root starts PostgreSQL, Keycloak, and Vault.
- Realms are identical on every machine.
- Works the same on macOS, Linux, and Windows (no OS-specific scripts).

**Non-Goals:**
- Production hardening (TLS, HA, Vault auto-unseal). Recorded for a later change.
- Email (SMTP) for Keycloak; email verification is a later change.
- Databases for XBank services; each service change adds its own.
- API gateway.

## Decisions

### Repo layout
```
xbank-infrastructure/
+-- postgres/      docker-compose.yml, .env.example, init/ (creates keycloak DB)
+-- keycloak/      docker-compose.yml, .env.example, realms/xbank.json, realms/xbankDev.json
+-- vault/         docker-compose.yml, config/vault.hcl
+-- docker-compose.yml   (include: every tool)
+-- .gitignore, README.md
```
One folder per tool and a root compose with `include:`, so tools start and stop together. Every compose file sets `name: xbank`. All containers join the external network `xbank_network`; the README documents `docker network create xbank_network`.

### Versions
Pin exact image tags at apply time, using the latest stable versions: PostgreSQL 17, Keycloak 26.x, Vault 1.x. Never `latest`.

### Keycloak
- `start-dev` is not used; Keycloak runs `start` with HTTP enabled for local use (`KC_HTTP_ENABLED=true`, `KC_HOSTNAME_STRICT=false`), backed by PostgreSQL. Simpler than TLS now, close to prod behavior.
- Realms are imported with `--import-realm` from `keycloak/realms/`. Import skips realms that already exist, so edits to the JSON need a re-import (documented in the README). Alternative considered: keycloak-config-cli. Rejected for now as an extra tool.
- Default role: `USER` is added to each realm's `default-roles-<realm>` composite.
- Client `xbank-ms-auth`: confidential, `directAccessGrantsEnabled: true`, standard flow off, service account on, with `realm-management` roles `manage-users` and `view-users`. The JSON holds no secret; Keycloak generates it, and the README tells you to copy it into Vault.
- Telephone and FIN are stored as user attributes; the realm's user profile declares `phoneNumber` and `fin` so Keycloak accepts them. Uniqueness is enforced by ms-auth, not Keycloak.
- Admin console credentials come from `keycloak/.env` (`KC_BOOTSTRAP_ADMIN_USERNAME`/`PASSWORD`).

### Vault
- Server mode with `file` storage on a named volume, not dev mode, so secrets persist.
- First start: `vault operator init` and `unseal` by hand, then enable `kv-v2` at `secret/`. The unseal keys and root token are kept outside every repo. Steps are in the README.
- Alternative considered: dev mode. Rejected because secrets vanish on restart.

### Host ports
Keycloak `8180`, Vault `8200`, PostgreSQL `5432`. Configurable in each tool's `.env`.

## Risks / Trade-offs

- [Dev and prod realms share one Keycloak and database] → A bad realm change can affect prod. Accepted for this stage; separate instances are a later decision.
- [Vault unseal is manual] → After every restart Vault is sealed and services cannot read secrets. README documents unseal; auto-unseal is a later change.
- [Realm import does not update existing realms] → README gives the re-import command.
- [Unseal keys lost] → Vault data cannot be recovered. README says to store them in a password manager.
