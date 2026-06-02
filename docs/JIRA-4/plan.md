# JIRA-4: Create a GET /hello/world endpoint

## Requirements

Create a minimal Spring Boot microservice that exposes a single REST endpoint at `GET /hello/world` returning a simple greeting response. The implementation covers the full lifecycle of a microservice:

- **Scaffold** a new Spring Boot project using Gradle with Java 17
- **Implement** the `GET /hello/world` endpoint returning JSON `{"message": "Hello, World!"}`
- **Health check** endpoint (`/actuator/health`) for container orchestration and readiness probes
- **Docker support** — multi-stage Dockerfile and `.dockerignore`
- **CI pipeline** — GitHub Actions workflow for build, test, and Docker image build
- **Coding standards** — Java 17+, Spring Boot, Gradle, no Lombok, fail-fast principle

### Acceptance Criteria

1. `GET /hello/world` returns HTTP 200 with JSON body `{"message": "Hello, World!"}`
2. Health endpoint `GET /actuator/health` returns HTTP 200
3. Application builds successfully with `./gradlew build`
4. Application starts and the endpoint responds correctly via `docker compose up` or `java -jar`
5. GitHub Actions CI workflow runs tests and builds the Docker image on every push

## Implementation Approach

### Technology Stack

| Component | Choice | Rationale |
|-----------|--------|-----------|
| Language | Java 17 | Aligns with architecture standards; enables records, pattern matching, text blocks |
| Framework | Spring Boot 3.2.4 | Industry standard; integrates with Actuator, Web starter |
| Build | Gradle 8.5 (wrapper) | Reproducible builds; no system-wide Gradle required |
| Testing | JUnit 5 + Spring Boot Test | First-class Spring integration; `@WebMvcTest` for controller slice tests |
| Container | Multi-stage Docker (eclipse-temurin:17-jre-alpine) | Minimal final image (JRE-only); separates build from runtime |
| CI/CD | GitHub Actions | Required by architecture; validates → tests → builds Docker image |

### Key Design Decisions

1. **Spring Boot 3.2.4** — Latest stable Spring Boot 3.x line; uses Spring Framework 6.x with GraalVM native-image support path available if needed later
2. **Actuator health endpoint** — Exposes `/actuator/health` which is required by ECS Fargate for health checks and by Kubernetes for readiness/liveness probes
3. **No Lombok** — All code is explicit (records for DTOs, no hidden bytecode generation); aligns with coding standards prohibition
4. **Fail-fast startup** — Application will fail at startup if critical environment variables are missing (Postel's Law is not adopted)
5. **Modern Java features** — Uses `record` for the immutable `HelloWorldResponse` DTO instead of a verbose POJO
6. **Multi-stage Docker build** — Build stage uses `gradle:8.5-jdk17`, runtime stage uses `eclipse-temurin:17-jre-alpine` to minimise image size
7. **`@WebMvcTest` for controller tests** — Slice-testing approach that only loads web layer beans, avoiding full context startup for faster feedback
8. **docker-compose.yml** — Provides a convenient `docker compose up` path for local development and aligns with AC #4

## Files to Modify

### Project Scaffolding

| File | Action | Description |
|------|--------|-------------|
| `settings.gradle` | Create | Root project name set to `my-microservice` |
| `build.gradle` | Create | Spring Boot 3.2.4, Web + Actuator starters, JUnit 5 test dependency |
| `gradle/wrapper/gradle-wrapper.properties` | Create | Gradle 8.5 distribution URL |
| `gradlew` | Create | Unix Gradle wrapper script |
| `gradlew.bat` | Create | Windows Gradle wrapper script |
| `src/main/java/com/example/microservice/MicroserviceApplication.java` | Create | Main class annotated with `@SpringBootApplication` |
| `src/main/resources/application.yml` | Create | Server port 8080, Actuator health endpoint with `show-details: always` |
| `src/test/java/com/example/microservice/MicroserviceApplicationTests.java` | Create | Smoke test verifying Spring context loads |

### Endpoint Implementation

| File | Action | Description |
|------|--------|-------------|
| `src/main/java/com/example/microservice/controller/HelloWorldController.java` | Create | `@RestController` mapped to `/hello` with `@GetMapping("/world")` returning `HelloWorldResponse` |
| `src/main/java/com/example/microservice/controller/HelloWorldResponse.java` | Create | Java `record` with a single `String message` field (immutable, no Lombok) |
| `src/test/java/com/example/microservice/controller/HelloWorldControllerTest.java` | Create | `@WebMvcTest(HelloWorldController.class)` verifying HTTP 200, JSON content type, and `$.message` value |

### Containerisation

| File | Action | Description |
|------|--------|-------------|
| `Dockerfile` | Create | Multi-stage: `gradle:8.5-jdk17` build stage → `eclipse-temurin:17-jre-alpine` runtime stage; exposes port 8080 |
| `.dockerignore` | Create | Excludes `build/`, `.gradle/`, `.git/`, `*.md`, `.idea/`, `.env`, `gradlew.bat` |
| `docker-compose.yml` | Create | Single service definition referencing the built Docker image, mapping host port 8080 to container port 8080 |

### CI/CD

| File | Action | Description |
|------|--------|-------------|
| `.github/workflows/ci.yml` | Create | Triggered on push/PR to `main`; steps: Checkout → JDK 17 setup → `./gradlew build` → `docker build` |

### Documentation

| File | Action | Description |
|------|--------|-------------|
| `README.md` | Update | Add build, run, Docker, and endpoints sections |

## Testing Strategy

### Automated Tests

1. **MicroserviceApplicationTests** (smoke test)
   - `@SpringBootTest` — Verifies the full application context loads without errors
   - Fails fast if beans are misconfigured or dependencies are missing

2. **HelloWorldControllerTest** (`@WebMvcTest`)
   - Verifies `GET /hello/world` returns HTTP 200
   - Verifies response content type is `application/json`
   - Verifies response body `$.message` equals `"Hello, World!"`
   - Uses `MockMvc` for lightweight web layer testing (no full server start)

### Manual Verification

1. Build verification:
   ```bash
   ./gradlew build
   ```
   - Produces `build/libs/my-microservice-0.0.1-SNAPSHOT.jar`

2. Runtime verification (JAR):
   ```bash
   java -jar build/libs/my-microservice-0.0.1-SNAPSHOT.jar
   curl http://localhost:8080/hello/world
   curl http://localhost:8080/actuator/health
   ```

3. Runtime verification (Docker):
   ```bash
   docker compose up --build
   curl http://localhost:8080/hello/world
   curl http://localhost:8080/actuator/health
   ```

### CI Verification

- Push to `main` (or PR targeting `main`) triggers GitHub Actions workflow
- Workflow runs `./gradlew build` (which runs all tests)
- If tests pass, Docker image is built
- Pipeline fails on any test failure or build error

## Dependencies and Considerations

### External Dependencies

| Dependency | Version | Purpose |
|------------|---------|---------|
| `org.springframework.boot:spring-boot-starter-web` | 3.2.x | REST endpoints and embedded Tomcat |
| `org.springframework.boot:spring-boot-starter-actuator` | 3.2.x | Health check endpoint |
| `org.springframework.boot:spring-boot-starter-test` | 3.2.x | JUnit 5, Spring Test, MockMvc |

### Build Tool

- Gradle 8.5 via wrapper (`gradlew`) — no system install required
- Wrapper scripts (`gradlew` + `gradlew.bat`) committed to repository

### Infrastructure Considerations

- Application runs on port 8080 (configurable via `SERVER_PORT` environment variable)
- Health endpoint exposed at `/actuator/health` for ECS Fargate / Kubernetes probes
- No database or external service dependencies — fully self-contained
- No authentication/authorisation required for these endpoints

## Definition of Done

- [ ] `settings.gradle` and `build.gradle` created with Web, Actuator, and Test dependencies
- [ ] Gradle wrapper (`gradlew`, `gradlew.bat`, `gradle/wrapper/*`) committed
- [ ] `MicroserviceApplication.java` compiles and starts on port 8080
- [ ] `HelloWorldController` responds to `GET /hello/world` with HTTP 200 and `{"message": "Hello, World!"}`
- [ ] Actuator health endpoint returns HTTP 200 with `{"status":"UP"}`
- [ ] `MicroserviceApplicationTests` (context load) passes
- [ ] `HelloWorldControllerTest` (WebMvcTest) passes
- [ ] `./gradlew build` succeeds (compilation + tests)
- [ ] `Dockerfile` and `.dockerignore` created; `docker build` succeeds
- [ ] `docker-compose.yml` created; `docker compose up` starts the service on port 8080
- [ ] GitHub Actions workflow `.github/workflows/ci.yml` created and runs on push/PR to `main`
- [ ] All code follows coding standards: no Lombok, fail-fast principle, modern Java features (records)
- [ ] `README.md` updated with build, run, Docker, and endpoint documentation
