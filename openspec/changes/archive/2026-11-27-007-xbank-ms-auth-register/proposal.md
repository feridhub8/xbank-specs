# Proposal

## Why

XBank's frontend needs users to sign up. The frontend must never talk to Keycloak directly or hold a client secret, so ms-auth registers the user in Keycloak on its behalf. This is the first real feature of ms-auth; login, refresh, logout, and `me` follow as separate changes.

## What Changes

- `POST /api/v1/auth/register` with `fin`, `email`, `password`, `firstName`, `lastName`, `phoneNumber`. Returns `201` with the new user's profile and roles (no password, no tokens).
- Input validation: FIN 7 uppercase letters or digits, phone `+994` and 9 digits, valid email, names required, password policy (8+ characters, upper, lower, digit). Failures use the existing `VALIDATION_FAILED` format.
- FIN, email, and phone are unique. New error codes: `FIN_ALREADY_EXISTS`, `EMAIL_ALREADY_EXISTS`, `PHONE_ALREADY_EXISTS` (all `409`). Two registrations with the same phone at the same time leave exactly one user.
- `KeycloakClient` (ms-auth's first client): service-account token, create, search by phone, read, delete users, read roles. Keycloak being unreachable gives `503 UPSTREAM_UNAVAILABLE`.
- Keycloak URL, realm, and client ID per profile (`local` and `dev` use `xbankDev`, `prod` uses `xbank`).
- Unit tests and Testcontainers integration tests against a Keycloak that imports the real realm file.

Out of scope: login, refresh, logout, `me`, Spring Security (the endpoint is public; security arrives with the first protected endpoint), email verification.

## Capabilities

### New Capabilities

- `user-auth`: registering users and, in later changes, issuing, refreshing, revoking, and reading their tokens through XBank's own API.

### Modified Capabilities

None.

## Impact

- `xbank-ms-auth`: new `auth/` domain (controller, service, DTOs, client, mapper, exceptions), `ErrorCode` gets three codes, `KeycloakProperties` gets `url`, `realm`, `clientId`, profile files, `build.gradle` (RestClient, MapStruct, Testcontainers Keycloak), tests, README.
- Tests read `../xbank-infrastructure/keycloak/realms/xbankDev.json`.
- No change to `xbank-infrastructure`.
