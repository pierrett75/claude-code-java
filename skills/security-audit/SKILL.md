---
name: security-audit
description: Security rules and audit checklist for Java 25 / Spring Boot 4 / Spring Security 7 apps, mapped to OWASP Top 10:2025. Use when writing or reviewing auth, endpoints, queries, file handling, outbound HTTP, config or dependencies, before releases, or when asked about security or vulnerabilities.
---

# Security rules (Spring Boot 4)

Apply when writing code and when auditing. Conventions in spring-boot-patterns still apply (DTO records, constructor injection, `@ConfigurationProperties`).

## Access control (A01)
- Deny by default: the `SecurityFilterChain` ends with `anyRequest().authenticated()`; open endpoints are listed explicitly.
- `@EnableMethodSecurity` must be on, otherwise `@PreAuthorize` does nothing.
- Object-level checks in the service: load the resource, verify the current user owns it or has the role. Never trust IDs from the request. Return 404 (not 403) for resources the user may not know exist.
- CORS: explicit allowed origins; never `*` together with credentials.
- SSRF (merged into A01): outbound HTTP only to configured base URLs / host allowlist; never fetch a URL taken from user input without validating scheme and host and blocking internal/private ranges and metadata IPs.

## Misconfiguration (A02)
- Actuator: expose only `health` and `info` over the web (`management.endpoints.web.exposure.include=health,info`). Never expose `env`, `heapdump`, `configprops`, `loggers`, `threaddump` publicly; if needed, put them behind auth or a separate management port.
- Keep Spring Security default headers (HSTS, `nosniff`, frame `DENY`, `X-XSS-Protection: 0`). Add a CSP only for apps serving HTML. Don't set `X-XSS-Protection: 1`.
- CSRF stays enabled for session/cookie-based apps. Disable it only for stateless APIs authenticated with bearer tokens.
- `server.error.include-stacktrace=never`, no debug/devtools in production, no default credentials.

## Supply chain (A03)
- Versions come from the Spring Boot BOM; don't override managed versions without reason.
- Dependency scanning in CI (Dependabot/Renovate and/or OWASP dependency-check with an NVD API key), failing on high severity.
- Update dependencies deliberately, never with `mvn versions:use-latest-releases`. Review new dependencies before adding them.

## Cryptography and secrets (A04)
- Passwords: `PasswordEncoderFactories.createDelegatingPasswordEncoder()` (bcrypt by default, upgradable). Never MD5/SHA-1/plain SHA-256 for passwords.
- Tokens and IDs that must be unguessable: `SecureRandom` / UUID v4, never `Random`.
- Secrets only from env vars or a secret manager, bound via `@ConfigurationProperties`. Never in code, `application.yml` or logs.
- `.gitignore` lists concrete files: `.env`, `application-local.yml`, `*.pem`, `*.key`, `*.p12`, `*.jks`. No broad patterns like `*secret*` that hide source files.

## Injection (A05)
- Queries: derived methods, JPQL with named parameters, Criteria API. No string concatenation in JPQL/SQL, ever.
- Dynamic sort/column names: map from an allowlist; never pass user input into `ORDER BY` or column names.
- Validation: Bean Validation on request records, allowlist patterns (`@Pattern`) for formats. No blocklist regexes for "dangerous" content.
- XSS: JSON APIs return `application/json`. Thymeleaf uses `th:text`, never `th:utext` for user content. User-supplied rich HTML is cleaned with OWASP Java HTML Sanitizer.
- No `Runtime.exec` / `ProcessBuilder` with user input.

## Design and mass assignment (A06)
- Never bind request bodies to entities. Request DTOs contain only fields the client may set: no `id`, `role`, `ownerId`, `status`, audit fields.
- Rate-limit login, password reset and other expensive endpoints.

## Authentication (A07)
- JWT/OAuth2: use `spring-boot-starter-oauth2-resource-server`; validate issuer and audience; no hand-written JWT parsing; nothing secret in token claims.
- Passwords: minimum length (12+), max length 64+ allowed, check against breached passwords; no composition rules (upper/lower/digit).
- Generic login error messages; log failed attempts; lock out or throttle repeated failures.

## Data integrity and deserialization (A08)
- Never `ObjectInputStream` on untrusted data.
- Jackson 3: never enable default typing. For polymorphism use `@JsonTypeInfo(use = NAME)` + `@JsonSubTypes` on a sealed type. Don't replace the auto-configured mapper; customize it through Spring Boot properties/customizers.
- File uploads: size limits (`spring.servlet.multipart.max-file-size`), check type by content not extension, generate the stored filename server-side, store outside the web root.
- File paths from input: `base.resolve(name).normalize()` and verify it `startsWith(base)` (path traversal).

## Logging and alerting (A09)
- Log security events: login success/failure, access denied, privilege changes, with user id and IP.
- Never log passwords, tokens, session IDs, full request bodies or personal data.
- Use structured logging so user input can't inject fake log lines.

## Exceptional conditions (A10)
- Fail closed: if an authorization check or external call throws, deny the action.
- Never swallow exceptions (`catch (Exception e) {}`); handle or rethrow.
- Clients get generic error responses from the global exception handler; details go to logs only.

## Audit output
For each finding report: severity (critical/high/medium/low), file and line, OWASP 2025 category, why it is exploitable here, and the concrete fix. List critical/high first. If nothing is found in a category, say so briefly instead of padding.
