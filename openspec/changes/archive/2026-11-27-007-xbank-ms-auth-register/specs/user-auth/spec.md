# Spec Delta

## Purpose

Lets XBank's own frontend register users and manage their sign-in through XBank's API, without ever talking to the identity provider directly.

## ADDED Requirements

### Requirement: Register a user
`POST /api/v1/auth/register` SHALL create a user from `fin`, `email`, `password`, `firstName`, `lastName`, and `phoneNumber`, give the user role `USER`, and return `201` with `fin`, `email`, `firstName`, `lastName`, `phoneNumber`, and `roles`. The response SHALL NOT contain the password or any token.

#### Scenario: Successful registration
- **WHEN** a valid, unused FIN, email, and phone are submitted
- **THEN** the response is `201` with the submitted `fin`, `email`, `firstName`, `lastName`, and `phoneNumber`, and `roles` equal to `["USER"]`

#### Scenario: FIN returned in uppercase
- **WHEN** a user registers with FIN `AB12C3D`
- **THEN** the response `fin` is `AB12C3D`

### Requirement: Registration input validation
Registration SHALL return `400` with code `VALIDATION_FAILED`, listing every invalid field, when: FIN is not 7 uppercase letters or digits; phone is not `+994` followed by 9 digits; email is not a valid address; first or last name is empty; password is shorter than 8 characters or lacks an uppercase letter, a lowercase letter, or a digit.

#### Scenario: Several invalid fields
- **WHEN** registration is submitted with FIN `ab12` and phone `0501234567`
- **THEN** the response is `400`, code `VALIDATION_FAILED`, and `errors` contains entries for `fin` and `phoneNumber`

#### Scenario: Weak password
- **WHEN** registration is submitted with password `password`
- **THEN** the response is `400`, code `VALIDATION_FAILED`, with an entry for `password`

### Requirement: FIN, email, and phone are unique
Registration SHALL fail with `409` when the FIN, email, or phone already belongs to a user, with code `FIN_ALREADY_EXISTS`, `EMAIL_ALREADY_EXISTS`, or `PHONE_ALREADY_EXISTS`. No two users SHALL remain with the same FIN, email, or phone.

#### Scenario: Duplicate FIN
- **WHEN** a user with FIN `AB12C3D` exists and another registration uses `AB12C3D`
- **THEN** the response is `409` with code `FIN_ALREADY_EXISTS`

#### Scenario: Duplicate email
- **WHEN** a user with email `user@xbank.test` exists and another registration uses it
- **THEN** the response is `409` with code `EMAIL_ALREADY_EXISTS`

#### Scenario: Duplicate phone
- **WHEN** a user with phone `+994501234567` exists and another registration uses it
- **THEN** the response is `409` with code `PHONE_ALREADY_EXISTS`

#### Scenario: Concurrent registrations with the same phone
- **WHEN** two registrations with different FINs and emails but the same phone run at the same time
- **THEN** exactly one returns `201`, the other returns `409` with code `PHONE_ALREADY_EXISTS`, and one user with that phone exists

### Requirement: Identity provider unavailable
When Keycloak cannot be reached, registration SHALL return `503` with code `UPSTREAM_UNAVAILABLE` and SHALL NOT leave a partly created user.

#### Scenario: Keycloak down
- **WHEN** registration is submitted while Keycloak is unreachable
- **THEN** the response is `503` with code `UPSTREAM_UNAVAILABLE`
