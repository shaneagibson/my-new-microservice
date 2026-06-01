# my-new-microservice

A Spring Boot microservice that exposes a greeting endpoint.

## Building

```bash
./gradlew build
```

## Running

```bash
java -jar build/libs/my-microservice-0.0.1-SNAPSHOT.jar
```

## Docker

```bash
docker build -t my-microservice .
docker run -p 8080:8080 my-microservice
```

## Endpoints

- `GET /hello/world` — Returns a greeting
- `GET /actuator/health` — Health check
