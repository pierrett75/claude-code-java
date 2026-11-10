---
name: spring-boot-patterns
description: My Spring Boot conventions. Use when creating or reviewing controllers, services, repositories, DTOs, exception handling, configuration or tests in a Spring Boot 4.x project.
---

# Spring Boot conventions

Follow these rules. Where a rule is not listed, use standard Spring Boot practice.

## Stack
- Spring Boot 4.1.x, Java 25. Lombok and MapStruct are on the classpath.
- Use Spring Boot 4 / Spring Framework 7 APIs and packages; don't fall back to Boot 3 examples (e.g., Jackson 3 uses `tools.jackson.databind.json.JsonMapper` instead of `com.fasterxml.jackson.databind.ObjectMapper`, while annotations stay in `com.fasterxml.jackson.annotation`).

## Layering
- Controller -> Service -> Repository. Controllers never touch repositories.
- Package structure uses clear subpackages (`controller`, `service`, `service/impl`, `repository`, `dto`, `entity`, `mapper`, `exception`).
- Controllers handle HTTP only: `@Valid` on request bodies, return `ResponseEntity`, no business logic.
- REST paths: `/api/v1/<plural-noun>`. POST returns 201 + `Location` header, DELETE returns 204.
- Never return JPA entities from controllers; always map to response DTOs.

## Injection
- Constructor injection only (`private final` fields + `@RequiredArgsConstructor`). No field `@Autowired`.

## Entities
- Use `@Entity`, `@Table`, `@Getter`, `@Setter`, `@NoArgsConstructor` (never `@Data` on JPA entities, as generated `equals`/`hashCode`/`toString` break lazy loading and collections).
- Implement `equals` and `hashCode` based on the database ID (handling null IDs for transient instances), and exclude lazy associations from `toString`.

## Services
- `@Transactional(readOnly = true)` on the class, plain `@Transactional` on write methods.
- Every service is an interface (`UserService`) plus an implementation (`UserServiceImpl` in `service/impl`). Controllers and other services depend on the interface only.
- Throw domain exceptions (`ResourceNotFoundException`, `BusinessException`), never generic `RuntimeException`.

## DTOs and mapping
- DTOs are Java records with Bean Validation annotations on request records.
- Record accessors are `id()`, not `getId()`.
- Use MapStruct (`@Mapper(componentModel = "spring")`); ignore `id` and audit fields when mapping request -> entity.

## Repositories
- Extend `JpaRepository`; no `@Repository` annotation needed.
- Prefer derived queries, then JPQL `@Query`; native SQL only when unavoidable.
- `Optional` for single results, `existsBy...` for existence checks.

## Exceptions
- One `@RestControllerAdvice` returns our custom `ErrorResponse(String code, String message)` record (do not confuse with `org.springframework.web.ErrorResponse`).
- Every custom exception MUST have a handler there (e.g. `BusinessException` -> 400/409 with its `code`).
- Handle `MethodArgumentNotValidException` -> 400 with field errors; catch-all `Exception` -> 500 with a generic message, logged at error level.

## Configuration
- Secrets only via environment variables (`${DB_PASSWORD}`), never committed.
- `spring.jpa.hibernate.ddl-auto: validate` outside local dev; schema changes go through Flyway/Liquibase.
- Typed config as records with `@ConfigurationProperties(prefix = "...")` + `@Validated`, enabled with `@ConfigurationPropertiesScan`.

## Tests
- Controller slice: `@WebMvcTest` (`org.springframework.boot.webmvc.test.autoconfigure.WebMvcTest` from `spring-boot-starter-webmvc-test`) + `@MockitoBean` (`org.springframework.test.context.bean.override.mockito.MockitoBean`, NOT the removed `@MockBean`).
- Service: test the `...Impl` class with JUnit + Mockito (`@ExtendWith(MockitoExtension.class)`).
- Integration: `@SpringBootTest` + `@ActiveProfiles("test")`. Real infrastructure we own (DB, Kafka, Redis) via Testcontainers; external systems (SMTP, payment, third-party APIs) replaced by stub beans.
- Stub beans live only in `src/test/java`, as `@TestConfiguration` imported into the test or as `@Component @Profile("test")`. A stub that records calls (e.g. `RecordingEmailSender` keeping a list of sent mails) is preferred over a mock when the test asserts on what was sent.
- Never put test-only beans or `@Profile("test")` classes in `src/main`.
- Testcontainers: the container needs `@ServiceConnection` so the datasource actually points at it:

```java
@Container
@ServiceConnection
static PostgreSQLContainer postgres = new PostgreSQLContainer("postgres:17"); // Testcontainers 2.x: org.testcontainers.postgresql
```
