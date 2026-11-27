# service-health Specification

## Purpose

Defines how every XBank service shows that it is running and which port it listens on, so people and tools can check any service the same way.

## Requirements

### Requirement: Health endpoint
Every XBank service SHALL answer `GET /actuator/health` with `200` and `{"status":"UP"}` while it is running. The endpoint SHALL need no authentication and SHALL expose no other actuator endpoints.

#### Scenario: Service is up
- **WHEN** the service has started
- **THEN** `GET /actuator/health` returns `200` with `status` `UP`

#### Scenario: Other actuator endpoints are hidden
- **WHEN** `GET /actuator/env` is called
- **THEN** the response is `404`

### Requirement: Local port
In the `local` profile a service SHALL listen on host port `90xx`, where `xx` is its service number. For `xbank-ms-auth` (number `01`) that is `9001`.

#### Scenario: ms-auth starts locally
- **WHEN** `xbank-ms-auth` starts with profile `local`
- **THEN** it listens on port `9001`
