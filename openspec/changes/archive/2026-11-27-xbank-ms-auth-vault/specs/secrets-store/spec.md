# Spec Delta

## ADDED Requirements

### Requirement: Services load their secrets at startup
A service SHALL read its secrets from `secret/xbank/<env>/<service>` at startup, signing in with its AppRole `<service>-<env>`, and expose each key as configuration. For `xbank-ms-auth` the key `keycloak-client-secret` SHALL become `xbank.keycloak.client-secret`.

#### Scenario: Secret is loaded
- **WHEN** `secret/xbank/local/ms-auth` holds `keycloak-client-secret` and ms-auth starts with profile `local` and valid `ms-auth-local` credentials
- **THEN** `xbank.keycloak.client-secret` has that value

### Requirement: No start without secrets
A service SHALL NOT start when Vault cannot be reached, rejects its AppRole credentials, or its required secret is missing.

#### Scenario: Wrong AppRole credentials
- **WHEN** ms-auth starts with a secret ID that Vault does not accept
- **THEN** startup fails

#### Scenario: Secret missing
- **WHEN** ms-auth starts with valid credentials but `keycloak-client-secret` is not in its path
- **THEN** startup fails

### Requirement: AppRole credentials stay out of repos
A service's AppRole role ID and secret ID SHALL come from untracked environment files (`.env`, `.env.dev`, `.env.prod`); the repo SHALL only hold `.env.example` with empty values.

#### Scenario: Example file
- **WHEN** `.env.example` is read
- **THEN** it lists `VAULT_ROLE_ID` and `VAULT_SECRET_ID` with empty values
