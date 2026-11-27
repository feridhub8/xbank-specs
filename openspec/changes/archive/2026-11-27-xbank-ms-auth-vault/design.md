# Design

## Context

See proposal.md. `xbank-ms-auth` (2 commits) has webmvc + actuator, profiles `local` (9001), `dev`/`prod` (8080 in the container), a Dockerfile, two compose files, and tests `HealthIT`, `ProfilesTest`, `ContainerIT`, `ComposeFileTest`. Vault 1.21.4 runs with AppRoles `ms-auth-{local,dev,prod}` and is empty under `secret/`. Spring Boot 4.2.0 works with Spring Cloud 2026.0.0 (checked on start.spring.io).

## Goals / Non-Goals

**Goals:**
- The secret arrives as a normal Spring setting; no code reads Vault directly.
- Every test still runs with `./gradlew test` and only Docker.

**Non-Goals:**
- Using the secret (register change), Keycloak URLs and realm settings (register change), secret rotation.

## Decisions

### Loading
- `spring.config.import: vault://secret/xbank/<env>/ms-auth` in each profile file, so each environment reads only its own path. `spring.cloud.vault.authentication: APPROLE` with `role-id: ${VAULT_ROLE_ID}` and `secret-id: ${VAULT_SECRET_ID}` in `application.yml`. Vault URI per profile: local `${VAULT_URL:http://localhost:8200}`, dev and prod `${VAULT_URL:http://vault:8200}`.
- `spring.cloud.vault.fail-fast: true`, so an unreachable or denying Vault stops startup instead of starting without secrets.
- KV v2 backend settings stay on (`spring.cloud.vault.kv.enabled: true`, `backend: secret`): the explicit `vault://secret/...` path needs them to resolve the v2 data path. Only the explicit path is read; the generic contexts are not used. (Changed during apply: the first draft turned KV off.)
- `KeycloakProperties` (`xbank.keycloak`, `@Validated`, `clientSecret` `@NotBlank`) in `common/config/v1`. `application.yml` maps `xbank.keycloak.client-secret: ${keycloak-client-secret:}`; the empty default matters, because an unresolved placeholder is otherwise bound as its literal text and passes `@NotBlank`. A missing secret then fails validation at startup. Lombok and validation starter are added here, because this is the first class that needs them.

### Credentials
- Local: `optional:file:./.env[.properties]` is imported in `application.yml`, not next to `vault://` in `application-local.yml`. Imports listed side by side are resolved together, so the Vault login ran before the `.env` values existed and failed with "invalid role or secret ID". (Found during apply; covered by `LocalEnvFileIT`.) Containers have no `.env` (`.dockerignore`), so the optional import is a no-op there.
- Dev and prod: compose `env_file: .env.dev` / `.env.prod`.
- Values are created with the `admin` login (README shows the commands). The secret ID never expires (`secret_id_ttl=0`, set by the infrastructure); rotating it is a later concern.

### Writing the secrets
Once, as `admin`: read each realm's `xbank-ms-auth` client secret through the Keycloak admin API and `vault kv put` it to the three paths. Not committed anywhere.

### Tests
Shared `support/TestVault` starts one `hashicorp/vault:1.21.4` dev-mode container per test run, runs `init-auth.sh` copied from `../xbank-infrastructure/vault/setup/` (Gradle copies it into test resources; path overridable with `-PinfrastructureDir`), writes a known test value to all three paths, and sets `VAULT_URL`, `VAULT_ROLE_ID`, `VAULT_SECRET_ID` as system properties for the AppRole of the environment under test.
- `VaultSecretsIT`: secret is loaded into `xbank.keycloak.client-secret`; startup fails with a wrong secret ID; startup fails when the key is missing (a separate path written without the key, selected through an overridden import).
- `HealthIT`, `ProfilesTest`: use `TestVault`; otherwise unchanged.
- `ContainerIT`: the image and the Vault container share a Testcontainers network; Vault gets the alias `vault`, so the dev and prod profiles reach it at `http://vault:8200` exactly as in compose.
- `ComposeFileTest`: also checks `env_file` (`.env.dev`, `.env.prod`).
- `EnvExampleTest`: `.env.example` has exactly `VAULT_ROLE_ID=` and `VAULT_SECRET_ID=`.
- `LocalEnvFileIT` (added during apply): runs the built jar as its own process in a folder holding only a `.env`, with no Vault credentials in its environment, and checks health is `UP`. The other tests pass credentials as system properties, so they could not catch the `.env` import order. `test` depends on `bootJar`.

## Risks / Trade-offs

- [Every test now needs a Vault container] → One shared container per test run; it starts in about a second.
- [Tests depend on the sibling infrastructure repo for `init-auth.sh`] → That is the real setup, so the test proves the policies work; the path is overridable.
- [Vault sealed after a restart stops ms-auth from starting] → Intended (`fail-fast`); README says to unseal first.
