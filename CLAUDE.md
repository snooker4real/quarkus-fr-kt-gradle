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
- **quarkus-rest** + **quarkus-rest-jackson** for JAX-RS/Jakarta REST endpoints with JSON
- **quarkus-arc** for CDI dependency injection
- Tests use `@QuarkusTest` + REST-Assured; native tests use `@QuarkusIntegrationTest`

## Key Extensions

| Extension | Dependency |
|---|---|
| REST + JSON | `quarkus-rest` + `quarkus-rest-jackson` |
| Panache ORM (Kotlin) | `quarkus-hibernate-orm-panache-kotlin` |
| PostgreSQL | `quarkus-jdbc-postgresql` |
| H2 (tests) | `quarkus-jdbc-h2` |
| Validation | `quarkus-hibernate-validator` |
| OpenAPI / Swagger UI | `quarkus-smallrye-openapi` |

> **JSON:** Always use `quarkus-rest-jackson` (or `quarkus-rest-jsonb`), not the standalone `quarkus-jsonb`. Without the REST-integrated variant, Quarkus falls back to `toString()` on response objects.

## Persistence with Panache

Two patterns — choose one per entity:

**Active record** (entity has DB methods):
```kotlin
@Entity
class Person : PanacheEntity() {
    lateinit var name: String
}
// usage: Person.findById(1), Person.listAll(), instance.persist()
```

**Repository** (separate class):
```kotlin
@Entity
class Person : PanacheEntityBase() { @Id var id: Long = 0; lateinit var name: String }

@ApplicationScoped
class PersonRepository : PanacheRepository<Person>
```

In dev mode, **Dev Services** auto-starts a PostgreSQL container — no local DB setup needed. Configure `application.properties` for production:
```properties
quarkus.datasource.db-kind=postgresql
quarkus.datasource.jdbc.url=jdbc:postgresql://localhost:5432/mydb
quarkus.datasource.username=myuser
quarkus.datasource.password=mypassword
quarkus.hibernate-orm.database.generation=update
```

## Architecture

REST resources live under `src/main/kotlin/com/snooker4real/` and are annotated with `@Path`. The `allOpen` Gradle plugin makes Quarkus-annotated classes open automatically (required for `@Path`, `@ApplicationScoped`, `@Entity`, `@QuarkusTest`), so Kotlin classes don't need explicit `open` modifiers.

Native test sources are in `src/native-test/` and integration test classes typically extend their JVM counterpart (see `GreetingResourceIT`).

## Docker

Four Dockerfiles in `src/main/docker/`:
- `Dockerfile.jvm` — standard JVM image (after `./gradlew build`)
- `Dockerfile.native` — GraalVM native (after native build)
- `Dockerfile.native-micro` — minimal native image
- `Dockerfile.legacy-jar` — legacy JAR packaging

All images expose port 8080.