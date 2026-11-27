# Spec Delta

## ADDED Requirements

### Requirement: FIN is the username
In both realms a user's username SHALL be their FIN. Keycloak SHALL reject a second user with the same FIN. Email SHALL stay unique per realm and SHALL be usable to log in.

#### Scenario: Duplicate FIN rejected
- **WHEN** a user with FIN `AB12C3D` exists and another user is created with username `AB12C3D`
- **THEN** Keycloak rejects it with `409`

#### Scenario: Login with email
- **WHEN** a user whose username is a FIN requests a token with their email and password
- **THEN** Keycloak issues an access token

#### Scenario: Email not used as username
- **WHEN** a user is created with username `AB12C3D` and email `user@xbank.test`
- **THEN** the stored username is the FIN, not the email
