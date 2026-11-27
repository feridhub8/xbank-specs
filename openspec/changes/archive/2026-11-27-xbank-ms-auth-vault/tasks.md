# Tasks

## 1. Test Vault

- [x] 1.1 Add the Spring Cloud BOM (2026.0.0), `testcontainers-vault`, the Gradle copy of `init-auth.sh` into test resources, and `support/TestVault` (dev-mode Vault, runs `init-auth.sh`, writes a test secret, sets system properties); verify with `TestVaultTest` that AppRole `ms-auth-local` can read its path in the test container

## 2. Load the secret

- [x] 2.1 Add `spring-cloud-starter-vault-config`, Lombok, the validation starter, `KeycloakProperties`, the Vault settings in `application.yml`, and the `vault://` imports in all three profiles (plus `.env` in local); move `HealthIT` and `ProfilesTest` onto `TestVault`; add `VaultSecretsIT` (secret loaded, wrong secret ID fails, missing key fails); verify `./gradlew test` passes except `ContainerIT`

## 3. Containers and compose

- [x] 3.1 Put Vault and the image on one Testcontainers network in `ContainerIT`; add `env_file` to both compose files and to `ComposeFileTest`; add `.env.example` and `EnvExampleTest`; verify `./gradlew test` passes in full

## 4. Real environment

- [x] 4.1 As Vault `admin`, write the realms' client secrets to `secret/xbank/{local,dev,prod}/ms-auth`, and create `.env`, `.env.dev`, `.env.prod` with AppRole credentials; verify `./gradlew bootRun` (local) and `docker compose -f docker-compose.dev.yml up -d --build` (dev) both answer `UP` on `/actuator/health` on 9001 and 9101, then stop them

## 5. Docs

- [x] 5.1 Update `README.md`: configuration files, getting AppRole credentials, Vault must be unsealed; verify the commands match the file names and AppRole names checked by the tests

## Workflow follow-up

- Stop for review of `xbank-ms-auth`; commit and push after approval.
- Archive this change, then commit `xbank-specs`.
