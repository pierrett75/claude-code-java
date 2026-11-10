---
name: solid-principles
description: My SOLID rules for Java 25 / Spring Boot 4 class design. Use when a class does too much, a type switch keeps growing, inheritance misbehaves, an interface is too fat, or a dependency points the wrong way; and for SOLID reviews. For naming, duplication and method length use clean-code; for layering use spring-boot-patterns.
---

# SOLID rules

Use SOLID to fix real problems in the code at hand, not to add abstractions in advance. Project conventions in spring-boot-patterns win over generic SOLID advice.

## S - Single Responsibility
- A class has one reason to change. Test: describe it in one sentence without "and".
- Signs of violation: unrelated fields used by different method groups, imports from many domains, names like `...Manager` / `...AndSomething`.
- Fix by extracting focused collaborators (e.g. `WelcomeEmailSender`, `UserAuditLogger`) and injecting them into the service.
- Side effects that are not part of the core operation (emails, audit, notifications) go through Spring events, published from the service and handled with `@TransactionalEventListener` (after commit), not called inline.
- Validation stays in Bean Validation on request records, not in a hand-written validator class.

## O - Open/Closed
- When a `switch` / `if-else` on a type or status keeps growing with new business variants, replace it with a strategy interface.
- In Spring, implementations are beans; inject them as `List<Strategy>` or `Map<String, Strategy>` and pick by key. Adding a variant = adding a class.
- A `switch` over a sealed interface or a stable enum is fine; don't introduce strategies for 2-3 values that never change.
- Money is `BigDecimal`, never `double`.

## L - Liskov Substitution
- A subtype must work anywhere its parent does: no stronger preconditions, no weaker guarantees, no `UnsupportedOperationException`, no surprise `null`.
- `instanceof` checks before calling a method signal a broken hierarchy.
- Prefer composition, interfaces and immutable records over class inheritance; use `sealed` interfaces for closed type families.

## I - Interface Segregation
- Clients depend only on methods they use. Signs of violation: empty or throwing implementations, very different subsets used by different clients.
- Applies to my own domain interfaces (services, ports, strategies). Spring Data repositories extending `JpaRepository` are NOT split into read/write interfaces.

## D - Dependency Inversion
- Business code never creates its own dependencies with `new` (except value objects, records, DTOs).
- Depend on interfaces for services and infrastructure ports (`NotificationSender`, `PaymentGateway`); infrastructure classes implement them.
- Constructor injection only (`private final` + `@RequiredArgsConstructor`).
- Pick implementations with config (`@Profile`, `@ConditionalOnProperty`), not if-statements.
- Test doubles never live in `src/main`. Test strategy is in spring-boot-patterns.

## Review checklist
| Principle | Question |
|-----------|----------|
| SRP | More than one reason to change? |
| OCP | Will the next variant force editing this class? |
| LSP | Any subtype that can't fully replace its parent? |
| ISP | Any empty or throwing implementations? |
| DIP | Any `new` of a service/infrastructure class in business code? |

Report each violation with: where, why it matters here, and the smallest refactoring that fixes it.
