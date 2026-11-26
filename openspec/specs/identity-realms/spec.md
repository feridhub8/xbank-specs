# identity-realms Specification

## Purpose

Provides the Keycloak realms that hold XBank users, their roles, and the clients backend services use, separated by environment.

## Requirements

### Requirement: Realm per environment
The identity provider SHALL expose realm `xbank` for the prod environment and realm `xbankDev` shared by the local and dev environments, both in one Keycloak instance.

#### Scenario: Prod uses the xbank realm
- **WHEN** a service runs in the prod environment
- **THEN** it authenticates users against realm `xbank`

#### Scenario: Local and dev share xbankDev
- **WHEN** a service runs in the local or dev environment
- **THEN** it authenticates users against realm `xbankDev`

### Requirement: Default USER role
Every user created in either realm SHALL receive the role `USER` automatically.

#### Scenario: New user gets USER
- **WHEN** a user is created in realm `xbankDev`
- **THEN** the user has role `USER` without any extra step

### Requirement: ADMIN role assigned by hand
Each realm SHALL define the role `ADMIN`, which is never assigned automatically and is granted only through the Keycloak admin console.

#### Scenario: New user is not admin
- **WHEN** a user is created in either realm
- **THEN** the user does not have role `ADMIN`

### Requirement: Password policy
Both realms SHALL reject passwords shorter than 8 characters or missing an uppercase letter, a lowercase letter, or a digit.

#### Scenario: Weak password rejected
- **WHEN** a user is created with password `password`
- **THEN** Keycloak rejects it with a password policy error

#### Scenario: Valid password accepted
- **WHEN** a user is created with password `Passw0rd`
- **THEN** Keycloak accepts it

### Requirement: Backend client for ms-auth
Each realm SHALL define a confidential client `xbank-ms-auth` that can obtain tokens with the password grant and can create and search users through the admin API. Its secret SHALL NOT be committed.

#### Scenario: Password grant works
- **WHEN** `xbank-ms-auth` requests a token with valid user credentials and its client secret
- **THEN** Keycloak returns an access token and a refresh token

#### Scenario: Client can create users
- **WHEN** `xbank-ms-auth` calls the admin API to create a user with its service account
- **THEN** the user is created

### Requirement: Reproducible realm setup
The realm configuration SHALL be committed and imported on startup, so a fresh machine gets identical realms without manual setup.

#### Scenario: Fresh start
- **WHEN** the infrastructure starts on a machine with empty volumes
- **THEN** realms `xbank` and `xbankDev` exist with their roles, policy, and client
