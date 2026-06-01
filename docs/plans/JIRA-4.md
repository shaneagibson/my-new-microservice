# JIRA-4: Create a GET /hello/world endpoint

## Requirements

Create a minimal Spring Boot microservice that exposes a single REST endpoint at `GET /hello/world` which returns a simple greeting response.

Based on JIRA-4 (Task, Priority: Medium):

- Scaffold a new Spring Boot project in this repository
- Implement a `GET /hello/world` endpoint that responds with a JSON payload (e.g., `{"message": "Hello, World!"}`)
- Include a health check endpoint for container orchestration/readiness probes
- Provide Docker support (Dockerfile and `.dockerignore`)
- Set up a basic CI pipeline via GitHub Actions
- Follow the project's coding standards (Java 17+, Spring Boot, Gradle, no Lombok, fail-fast principle)

### Acceptance Criteria

1. `GET /hello/world` returns HTTP 200 with JSON body `{"message": "Hello, World!"}`
2. Health endpoint `GET /actuator/health` returns HTTP 200
3. Application builds successfully with `./gradlew build`
4. Application starts and the endpoint responds correctly via `docker compose up` or `java -jar`
5. GitHub Actions CI workflow runs tests and builds the Docker image on every push

## Implementation Approach

### Technology Stack

- **Language:** Java 17 (using modern language features: records, streams, text blocks)
- **Framework:** Spring Boot 3.x (`spring-boot-starter-web` and `spring-boot-starter-actuator`)
- **Build:** Gradle with Gradle wrapper
- **Testing:** JUnit 5 + Spring Boot Test (`@WebMvcTest` for the controller)
- **Packaging:** Executable JAR via `bootJar`, then containerised with Docker
- **CI/CD:** GitHub Actions (validate → test → build Docker image)
- **Docker:** Multi-stage Dockerfile for minimal image size

### Key Design Decisions

1. **Spring Boot 3.x** — aligns with the architecture's Spring Boot standard and provides native GraalVM support path if needed later
2. **Gradle wrapper** — ensures reproducible builds without requiring a system-wide Gradle installation
3. **Actuator health endpoint** — provides a `/actuator/health` endpoint required for ECS Fargate health checks and Kubernetes readiness probes
4. **Fail-fast startup** — the application will fail at startup if critical environment variables are missing (aligns with coding standards)
5. **No Lombok** — all getters, constructors, and builders written explicitly per coding standards
6. **Multi-stage Docker build** — keeps the final image small by separating build and runtime stages

## Files to Modify

### Project Scaffolding

| File | Action | Description |
|------|--------|-------------|
| `build.gradle` | Create | Gradle build file with Spring Boot plugin, Spring Web, Actuator, and Test dependencies |
| `settings.gradle` | Create | Gradle settings defining the project name |
| `gradle/wrapper/gradle-wrapper.properties` | Create | Gradle wrapper configuration (Gradle 8.x) |
| `gradlew` | Create | Gradle wrapper script (Unix) |
| `gradlew.bat` | Create | Gradle wrapper script (Windows) |
| `src/main/java/com/example/microservice/MicroserviceApplication.java` | Create | Main application class with `@SpringBootApplication` |
| `src/main/resources/application.yml` | Create | Application configuration (server port, actuator settings, etc.) |
| `src/test/java/com/example/microservice/MicroserviceApplicationTests.java` | Create | Smoke test verifying application context loads |

### Endpoint Implementation

| File | Action | Description |
|------|--------|-------------|
| `src/main/java/com/example/microservice/controller/HelloWorldController.java` | Create | REST controller with `GET /hello/world` endpoint returning `{"message": "Hello, World!"}` |
| `src/main/java/com/example/microservice/controller/HelloWorldResponse.java` | Create | Immutable response record for the greeting payload |
| `src/test/java/com/example/microservice/controller/HelloWorldControllerTest.java` | Create | `@WebMvcTest` unit test for the controller |

### Containerisation

| File | Action | Description |
|------|--------|-------------|
| `Dockerfile` | Create | Multi-stage Docker build (Gradle build → distroless runtime) |
| `.dockerignore` | Create | Exclude build artifacts, IDE files, and local Gradle cache from Docker context |

### CI/CD

| File | Action | Description |
|------|--------|-------------|
| `.github/workflows/ci.yml` | Create | GitHub Actions workflow: build, test, lint, and publish Docker image |

### Documentation (if needed)

| File | Action | Description |
|------|--------|-------------|
| `README.md` | Update | Update with build/run instructions (already exists) |

## Testing Strategy

### Unit/Integration Tests

1. **HelloWorldControllerTest** (`@WebMvcTest`)
   - Verify `GET /hello/world` returns HTTP 200
   - Verify response body contains `{"message": "Hello, World!"}`
   - Verify content type is `application/json`

2. **MicroserviceApplicationTests** (smoke test)
   - Verify the Spring application context loads without errors

### Manual Verification

1. Run `./gradlew build` — should produce a runnable JAR
2. Run `java -jar build/libs/*.jar` — application should start on port 8080
3. `curl http://localhost:8080/hello/world` — should return `{"message":"Hello, World!"}`
4. `curl http://localhost:8080/actuator/health` — should return `{"status":"UP"}`
5. `docker build -t my-microservice . && docker run -p 8080:8080 my-microservice` — should work

## Definition of Done

- [ ] `build.gradle` and `settings.gradle` created with correct dependencies
- [ ] Gradle wrapper generated (`gradlew`, `gradlew.bat`, `gradle/wrapper/`)
- [ ] `MicroserviceApplication.java` compiles and starts
- [ ] `HelloWorldController` responds correctly to `GET /hello/world`
- [ ] Actuator health endpoint returns `UP`
- [ ] Unit tests pass (`./gradlew test`)
- [ ] Full build succeeds (`./gradlew build`)
- [ ] `Dockerfile` and `.dockerignore` created and image builds
- [ ] GitHub Actions workflow `.github/workflows/ci.yml` created
- [ ] All code follows coding standards (no Lombok, fail-fast, modern Java features)
- [ ] JIRA ticket updated with comment confirming implementation plan created
