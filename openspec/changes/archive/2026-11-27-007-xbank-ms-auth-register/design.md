# Design

## Context

See proposal.md and `specs/user-auth`. `xbank-ms-auth` has the health endpoint, Vault loading (`KeycloakProperties.clientSecret`), and the ProblemDetail handler (`common/exception/v1`). No Spring Security. Keycloak 26.8.2 realms use FIN as username (stored lowercase), unique email, password policy, default role `USER`, and client `xbank-ms-auth` with a service account (`manage-users`, `view-users`, `query-users`) and the `kc-admin-api` scope. The realm's user profile still declares a `fin` attribute from before FIN became the username; ms-auth never writes it (a later infrastructure cleanup can remove it).

## Goals / Non-Goals

**Goals:**
- Register with the uniqueness guarantees in the spec, including concurrent requests.
- The package layout from `config.yaml`, used for the first time with real code.

**Non-Goals:**
- Tokens (login change), Spring Security, the `urn:` question for the ProblemDetail `type` (separate change).

## Decisions

### Packages
```
com/xbank/ms/auth/
+-- common/config/v1/      KeycloakProperties (+ url, realm, clientId), RestClientConfig
+-- common/exception/v1/   ErrorCode (+ FIN/EMAIL/PHONE_ALREADY_EXISTS)
+-- auth/controller/v1/    AuthController
+-- auth/service/v1/       AuthService
+-- auth/dto/v1/           RegisterRequest, UserResponse
+-- auth/client/v1/        KeycloakClient, KeycloakUser, KeycloakTokens
+-- auth/mapper/v1/        UserMapper (MapStruct)
+-- auth/exception/v1/     FinAlreadyExistsException, EmailAlreadyExistsException,
                           PhoneAlreadyExistsException, PasswordRejectedException
```
DTOs are regular classes with Lombok. `KeycloakUser` and `KeycloakTokens` mirror Keycloak's JSON and live with the client.

### Configuration
`xbank.keycloak.url` (local `${KEYCLOAK_URL:http://localhost:8180}`, dev and prod `${KEYCLOAK_URL:http://keycloak:8080}`), `realm` (`xbankDev` for local and dev, `xbank` for prod), `client-id: xbank-ms-auth` in `application.yml`. All `@NotBlank`. Startup does not call Keycloak, so existing tests and containers start without it.

### KeycloakClient
Spring `RestClient` (needs `spring-boot-starter-restclient` in Spring Boot 4) with 3 s connect and 10 s read timeouts.
- Service-account token: `client_credentials` grant, cached until 30 s before expiry, `synchronized`.
- `createUser`: `POST /admin/realms/{realm}/users`, returns the id from the `Location` header. `409` body mentions `email` → `EmailAlreadyExistsException`, otherwise `FinAlreadyExistsException`. `400` mentioning the password → `PasswordRejectedException` (`VALIDATION_FAILED` on `password`).
- `findByPhone`: `GET .../users?q=phoneNumber:<phone>&exact=true&briefRepresentation=false`.
- `getUser`, `deleteUser`, `realmRoles` (`.../role-mappings/realm/composite`).
- Connection errors and `5xx` → `UpstreamUnavailableException`.
Plain REST, not the Keycloak admin client library: fewer dependencies, no version lock to the server.

### Registration flow (AuthService)
1. Bean Validation on `RegisterRequest` (controller `@Valid`).
2. Search by phone; any match → `PHONE_ALREADY_EXISTS`.
3. Create the user (username = FIN, email, names, attribute `phoneNumber`, password credential, enabled). FIN and email uniqueness are enforced by Keycloak itself.
4. Search by phone again. If more than one user has it, keep the oldest (`createdTimestamp`, then id); if that is not the new user, delete the new user and return `PHONE_ALREADY_EXISTS`.
5. Read the user and its realm roles; return `UserResponse` with FIN uppercased and roles filtered to XBank's own (`USER`, `ADMIN`).
With two concurrent requests, both see two users at step 4 and both pick the same oldest one, so exactly one survives.

### Tests
- `TestKeycloak` (support): one `keycloak/keycloak:26.8.2` container per run via `testcontainers-keycloak` 4.4.1, importing `xbankDev.json` copied from `../xbank-infrastructure/keycloak/realms/` by Gradle. It sets the `xbank-ms-auth` client secret to `TestVault.CLIENT_SECRET` through the admin API, so Vault and Keycloak agree. Tests that need it set `xbank.keycloak.url` with `@DynamicPropertySource`.
- Isolation from the real Keycloak (found during apply): Spring caches application contexts by configuration, so a context started by an earlier test (default URL `http://localhost:8180`, the real local Keycloak) was reused by `RegisterIT`. `@DynamicPropertySource` gives Keycloak tests their own context, and `TestVault.start()` sets `KEYCLOAK_URL` to `http://127.0.0.1:1` for every other test. `RealKeycloakGuardTest` checks this.
- Unit: `RegisterRequestValidationTest` (each rule), `AuthServiceTest` with a mocked client (phone pre-check, rollback of the newer user, keeping the older one, mapping).
- Integration: `KeycloakClientIT` (create, exact phone search, delete, duplicate mapping, weak password), `RegisterIT` over real HTTP (every scenario in `specs/user-auth`, three rounds of the concurrent case), `KeycloakUnavailableTest` (client against a closed port → `UpstreamUnavailableException`).

## Risks / Trade-offs

- [Phone uniqueness is check, create, recheck, not a hard constraint] → Concurrent duplicates are removed at step 4; a crash between steps 3 and 4 could leave one. A database with a unique constraint is the fix if it ever matters.
- [FIN vs email on `409` is read from Keycloak's message text] → Covered by `KeycloakClientIT` against the pinned Keycloak version; an upgrade that changes the text fails the test.
- [Endpoint is public with no rate limit] → Accepted until the gateway exists.
- [Tests depend on the sibling infrastructure repo for the realm file] → Same as the Vault script; path overridable with `-PinfrastructureDir`.
