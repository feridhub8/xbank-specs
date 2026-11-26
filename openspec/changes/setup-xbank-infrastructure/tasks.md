# Tasks

## 1. Project decisions

- [x] 1.1 Update `xbank-specs/openspec/config.yaml` with environments (local/dev -> xbankDev, prod -> xbank), Vault path layout, and infrastructure tools (PostgreSQL, Keycloak, Vault); verify nothing infrastructure-related is left under "not decided yet"

## 2. Repo setup

- [x] 2.1 Create `XBank/xbank-infrastructure`, `git init`, add a complete `.gitignore` (OS, IDE, `.env`, logs, `graphify-out/`); verify `git status` shows only intended files
- [x] 2.2 Set up Graphify per the global rules (claude install, hook install, path fixes, first graph); verify `graphify hook status` and that committed files have no absolute paths

## 3. PostgreSQL

- [x] 3.1 Add `postgres/` (compose, `.env.example`, init script creating the `keycloak` database and user); verify `docker compose up -d` and `psql` lists the `keycloak` database

## 4. Keycloak

- [x] 4.1 Add `keycloak/` compose and `.env.example` using PostgreSQL; verify the admin console opens on `http://localhost:8180`
- [x] 4.2 Add `realms/xbankDev.json` and `realms/xbank.json` with roles, default `USER`, password policy, user profile attributes `phoneNumber` and `fin`, and client `xbank-ms-auth`; verify on empty volumes both realms are imported and the spec scenarios (weak password rejected, new user has `USER` only, password grant returns tokens) pass via the admin console and `curl`

## 5. Vault

- [x] 5.1 Add `vault/` compose and `config/vault.hcl` with file storage; verify init, unseal, enable `kv-v2` at `secret/`, write and read `secret/xbank/local/ms-auth`, restart, unseal, and read it again

## 6. Root and docs

- [x] 6.1 Add root `docker-compose.yml` including all tools; verify `docker compose up -d` and `down` from the root control every container
- [x] 6.2 Write `README.md`: network creation, start/stop, Keycloak re-import, Vault init/unseal, copying the client secret into Vault; verify by following it on empty volumes
- [x] 6.3 Check that no secret values are committed (spec: No secrets in repos); verify with `git grep` for passwords and secrets

## Workflow follow-up

- Commit and push `xbank-infrastructure` (create the GitHub repo) after approval.
- Archive this change in `xbank-specs`.
