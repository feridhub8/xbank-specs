# Spec Delta

## ADDED Requirements

### Requirement: Container port
Inside its container a service SHALL listen on port `8080` in the `dev` and `prod` profiles.

#### Scenario: dev profile in a container
- **WHEN** the service image runs with profile `dev`
- **THEN** `GET /actuator/health` on container port `8080` returns `200` with `status` `UP`

#### Scenario: prod profile in a container
- **WHEN** the service image runs with profile `prod`
- **THEN** `GET /actuator/health` on container port `8080` returns `200` with `status` `UP`

### Requirement: Dev and prod host ports
The `dev` environment SHALL publish the service on host port `91xx` and `prod` on `92xx`, where `xx` is the service number. For `xbank-ms-auth` that is `9101` (dev) and `9201` (prod), each mapped to container port `8080`.

#### Scenario: Dev compose file
- **WHEN** `docker-compose.dev.yml` is read
- **THEN** it runs profile `dev` and maps host port `9101` to `8080`

#### Scenario: Prod compose file
- **WHEN** `docker-compose.yml` is read
- **THEN** it runs profile `prod` and maps host port `9201` to `8080`

### Requirement: Runs in the shared Compose project
Every compose file of a service SHALL set the project name `xbank`, set no `container_name`, and attach the service to the external network `xbank_network`.

#### Scenario: Compose project and network
- **WHEN** either compose file of `xbank-ms-auth` is read
- **THEN** its `name` is `xbank`, no service sets `container_name`, and the service joins `xbank_network`, declared as external

### Requirement: Non-root container
The service process in the container SHALL NOT run as root.

#### Scenario: Container user
- **WHEN** the service image runs
- **THEN** the process user is not `root`
