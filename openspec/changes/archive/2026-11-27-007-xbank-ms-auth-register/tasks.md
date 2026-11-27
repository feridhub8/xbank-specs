# Tasks

## 1. Test Keycloak

- [x] 1.1 Add `testcontainers-keycloak` 4.4.1, the Gradle copy of `xbankDev.json` into test resources, and `support/TestKeycloak` (imports the realm, sets the client secret to `TestVault.CLIENT_SECRET`, sets `KEYCLOAK_URL`); verify with `TestKeycloakTest` that the client's service account gets a token with that secret

## 2. Keycloak client

- [x] 2.1 Add `url`, `realm`, `clientId` to `KeycloakProperties` and the three profiles, `spring-boot-starter-restclient`, `RestClientConfig`, `KeycloakUser`, `KeycloakTokens`, and `KeycloakClient` with the service-account token, `createUser`, `findByPhone`, `getUser`, `deleteUser`, `realmRoles`, and error mapping; add `FIN/EMAIL/PHONE_ALREADY_EXISTS` to `ErrorCode` and the four domain exceptions; verify `KeycloakClientIT` and `KeycloakUnavailableTest` pass and the existing suite still passes

## 3. Register

- [x] 3.1 Add MapStruct, `RegisterRequest` (validation), `UserResponse`, and `UserMapper`; verify `RegisterRequestValidationTest` passes
- [x] 3.2 Add `AuthService.register` (phone pre-check, create, recheck, rollback, mapping); verify `AuthServiceTest` passes
- [x] 3.3 Add `AuthController` with `POST /api/v1/auth/register`; verify `RegisterIT` covers every scenario in `specs/user-auth` and passes, and the full suite passes

## 4. Docs

- [x] 4.1 Update `README.md` (the register endpoint, request and response, error codes, `KEYCLOAK_URL`); verify the listed codes match `ErrorCode`

## Workflow follow-up

- Stop for review of `xbank-ms-auth`; commit and push after approval.
- Archive this change, then commit `xbank-specs`.
