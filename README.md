# quarkus-fr-kt-gradle

This project uses Quarkus, the Supersonic Subatomic Java Framework.

If you want to learn more about Quarkus, please visit its website: <https://quarkus.io/>.

## Running the application in dev mode

You can run your application in dev mode that enables live coding using:

```shell script
./gradlew quarkusDev
```

> **_NOTE:_**  Quarkus now ships with a Dev UI, which is available in dev mode only at <http://localhost:8080/q/dev/>.

## Packaging and running the application

The application can be packaged using:

```shell script
./gradlew build
```

It produces the `quarkus-run.jar` file in the `build/quarkus-app/` directory.
Be aware that it’s not an _über-jar_ as the dependencies are copied into the `build/quarkus-app/lib/` directory.

The application is now runnable using `java -jar build/quarkus-app/quarkus-run.jar`.

If you want to build an _über-jar_, execute the following command:

```shell script
./gradlew build -Dquarkus.package.jar.type=uber-jar
```

The application, packaged as an _über-jar_, is now runnable using `java -jar build/*-runner.jar`.

## Creating a native executable

You can create a native executable using:

```shell script
./gradlew build -Dquarkus.native.enabled=true
```

Or, if you don't have GraalVM installed, you can run the native executable build in a container using:

```shell script
./gradlew build -Dquarkus.native.enabled=true -Dquarkus.native.container-build=true
```

You can then execute your native executable with: `./build/quarkus-fr-kt-gradle-1.0.0-SNAPSHOT-runner`

If you want to learn more about building native executables, please consult <https://quarkus.io/guides/gradle-tooling>.

## Related Guides

- Kotlin ([guide](https://quarkus.io/guides/kotlin)): Write your services in Kotlin
- Quarkus REST + Jackson ([guide](https://quarkus.io/guides/rest#json-serialisation)): JSON serialization for REST endpoints
- Hibernate ORM with Panache for Kotlin ([guide](https://quarkus.io/guides/hibernate-orm-panache-kotlin)): Simplified ORM with active record and repository patterns
- Quarkus Dev Services ([guide](https://quarkus.io/guides/dev-services)): Zero-config databases and brokers in dev/test mode
- SmallRye OpenAPI ([guide](https://quarkus.io/guides/openapi-swaggerui)): Expose OpenAPI spec and Swagger UI at `/q/swagger-ui`
- Quarkus Validation ([guide](https://quarkus.io/guides/validation)): Bean Validation with `@Valid`, `@NotNull`, etc.

## Adding Persistence with Panache

Add to `build.gradle`:

```groovy
implementation 'io.quarkus:quarkus-hibernate-orm-panache-kotlin'
implementation 'io.quarkus:quarkus-jdbc-postgresql'   // or quarkus-jdbc-h2 for tests
```

Configure `application.properties`:

```properties
quarkus.datasource.db-kind=postgresql
quarkus.datasource.username=myuser
quarkus.datasource.password=mypassword
quarkus.datasource.jdbc.url=jdbc:postgresql://localhost:5432/mydb
quarkus.hibernate-orm.database.generation=update
```

Entity example (active record pattern):

```kotlin
@Entity
class Person : PanacheEntity() {
    lateinit var name: String
}
```

Repository example:

```kotlin
@ApplicationScoped
class PersonRepository : PanacheRepository<Person>
```

> In dev mode, Quarkus Dev Services automatically starts a PostgreSQL container — no local DB needed.
