# secrets-store Specification

## Purpose

Defines where XBank services keep their secrets per environment, so no secret ever lives in a repo or in plain configuration.

## Requirements

### Requirement: Secret path layout
Secrets SHALL be stored in Vault's KV v2 engine at `secret/xbank/<env>/<service>`, where `<env>` is `local`, `dev`, or `prod` and `<service>` is the repo name without the `xbank-` prefix.

#### Scenario: ms-auth local secrets
- **WHEN** `xbank-ms-auth` runs in the local environment
- **THEN** it reads its secrets from `secret/xbank/local/ms-auth`

### Requirement: Secrets survive restarts
Vault SHALL keep stored secrets across container restarts.

#### Scenario: Restart keeps secrets
- **WHEN** a secret is written, and the Vault container is restarted and unsealed
- **THEN** the secret is still readable

### Requirement: No secrets in repos
Repos SHALL contain no secret values: only `.env.example` files with placeholder values.

#### Scenario: Repo scan
- **WHEN** the committed files of `xbank-infrastructure` are inspected
- **THEN** no password, token, or client secret value appears in them

### Requirement: One AppRole per service and environment
Each service SHALL sign in to Vault with its own AppRole per environment, named `<service>-<env>` (for example `ms-auth-local`), whose policy allows reading only `secret/xbank/<env>/<service>`.

#### Scenario: Own path is not denied
- **WHEN** ms-auth signs in with AppRole `ms-auth-local` and reads `secret/xbank/local/ms-auth`
- **THEN** Vault does not deny the request

#### Scenario: Other environments are denied
- **WHEN** ms-auth signs in with AppRole `ms-auth-local` and reads `secret/xbank/prod/ms-auth`
- **THEN** Vault denies the request with `403`

### Requirement: People sign in with username and password
People SHALL sign in to Vault with the `userpass` method. A user `admin` SHALL exist whose token can manage secrets, policies, and auth methods.

#### Scenario: Admin login
- **WHEN** `admin` signs in with userpass and its password
- **THEN** Vault returns a token that can list policies and write a secret

### Requirement: Setup is repeatable
The Vault auth setup SHALL be safe to run again without errors or changing existing roles' IDs.

#### Scenario: Running the setup twice
- **WHEN** the setup runs on a Vault where it already ran
- **THEN** it finishes without errors and the role ID of `ms-auth-local` is unchanged

### Requirement: No standing root token
After the setup no token with the `root` policy SHALL remain valid. A root token is generated from the unseal key only when needed and revoked afterwards.

#### Scenario: No root tokens remain
- **WHEN** the setup is complete
- **THEN** no valid token in Vault has the `root` policy
