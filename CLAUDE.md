# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Development (live reload + Dev UI at http://localhost:8080/q/dev/)
./gradlew quarkusDev

# Build
./gradlew build

# Run tests
./gradlew test

# Run a single test class
./gradlew test --tests "com.snooker4real.GreetingResourceTest"

# Native executable (requires GraalVM, or use container-build flag)
./gradlew build -Dquarkus.native.enabled=true
./gradlew build -Dquarkus.native.enabled=true -Dquarkus.native.container-build=true

# Native integration tests
./gradlew integrationTest

# Über-jar (single file)
./gradlew build -Dquarkus.package.jar.type=uber-jar
```

## Tech Stack

- **Quarkus** 3.31.4 with **Kotlin** 2.3.0 and **Gradle** 9.3.1
- **Java 25** (source and target compatibility)
- **quarkus-rest** for JAX-RS/Jakarta REST endpoints
- **quarkus-arc** for CDI dependency injection
- Tests use `@QuarkusTest` + REST-Assured; native tests use `@QuarkusIntegrationTest`

## Architecture

The app follows standard Quarkus REST resource patterns. REST resources live under `src/main/kotlin/com/snooker4real/` and are annotated with `@Path`. The `allOpen` Gradle plugin makes Quarkus-annotated classes open automatically (required for `@Path`, `@ApplicationScoped`, `@Entity`, `@QuarkusTest`), so Kotlin classes don't need explicit `open` modifiers.

Native test sources are in `src/native-test/` and integration test classes typically extend their JVM counterpart (see `GreetingResourceIT`).

**Note:** `GreetingResourceTest` currently asserts `"Hello from Quarkus REST"` but `GreetingResource` returns `"Hello from Quarkus Kt"` — the test body is mismatched and will fail.

## Docker

Four Dockerfiles in `src/main/docker/`:
- `Dockerfile.jvm` — standard JVM image (after `./gradlew build`)
- `Dockerfile.native` — GraalVM native (after native build)
- `Dockerfile.native-micro` — minimal native image
- `Dockerfile.legacy-jar` — legacy JAR packaging

All images expose port 8080.