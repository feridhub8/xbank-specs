# Spec Delta

## Purpose

Defines where XBank services keep their secrets per environment, so no secret ever lives in a repo or in plain configuration.

## ADDED Requirements

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
