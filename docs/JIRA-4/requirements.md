# JIRA-4: Create a GET /hello/world endpoint

## Source
- JIRA URL: https://shaneagibson.atlassian.net/browse/JIRA-4
- Type: Task
- Priority: Medium
- Status: In Progress

## Description

Create a minimal Spring Boot microservice that exposes a single REST endpoint at `GET /hello/world` which returns a simple greeting response.

The implementation covers the full lifecycle of a microservice:

- **Scaffold** a new Spring Boot project in this repository
- **Implement** a `GET /hello/world` endpoint that responds with a JSON payload
- **Health check** endpoint for container orchestration / readiness probes
- **Docker support** — Dockerfile and `.dockerignore`
- **CI pipeline** — GitHub Actions workflow for build, test, and Docker image build
- **Coding standards** — Java 17+, Spring Boot, Gradle, no Lombok, fail-fast principle

## Acceptance Criteria

1. `GET /hello/world` returns HTTP 200 with JSON body `{"message": "Hello, World!"}`
2. Health endpoint `GET /actuator/health` returns HTTP 200
3. Application builds successfully with `./gradlew build`
4. Application starts and the endpoint responds correctly via `docker compose up` or `java -jar`
5. GitHub Actions CI workflow runs tests and builds the Docker image on every push

## Attachments

- No attachments on the JIRA ticket.

## Analysis Notes

- The JIRA ticket summary ("Create a GET /hello/world endpoint") is clear and descriptive.
- The JIRA ticket does not have a formal description field (null), but the requirements are fully elaborated in the existing implementation plan at `docs/plans/JIRA-4.md`.
- All acceptance criteria are specific, measurable, and testable.
- Technology choices (Java 17, Spring Boot 3.x, Gradle, Docker, GitHub Actions) are consistent with project coding standards.
- No ambiguous or contradictory requirements detected.
- No attachments were found on the ticket.
