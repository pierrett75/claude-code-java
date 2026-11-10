---
name: clean-code
description: My clean-code rules for Java 25 / Spring Boot 4 code. Use when writing, refactoring or reviewing code for readability - naming, method size, duplication, nesting, comments, magic values. For Spring layering and wiring, also follow spring-boot-patterns.
---

# Clean code rules

Apply these when writing or reviewing code. Fix what violates them; don't rewrite code that already follows them.

## Project conventions win
- Interface + Impl for every service is an intentional rule (see spring-boot-patterns), NOT a YAGNI violation. Don't suggest removing it.
- Repositories extend `JpaRepository`; don't replace them with hand-written minimal interfaces.
- Input validation belongs in Bean Validation annotations on request records (`@Valid`), not hand-written checks in controllers.

## Naming
- Names say what something is or does: `activeUsers`, `sendWelcomeEmail()`, `ShippingCalculator`. No `data`, `info`, `tmp`, `list`, `Manager`, `Helper`, `Utils` dumping grounds.
- Booleans start with `is` / `has` / `can` / `should`.
- A method's name must match its behaviour exactly (a method that trims and checks emptiness is `isBlank`, not `isEmpty`).

## Methods
- One job per method, one level of abstraction. If a method needs section comments, extract those sections.
- Guideline, not hard limit: ~20 lines, ~3-4 parameters. Above that, extract a method or introduce a parameter record.
- No boolean flag parameters that switch behaviour; split into two clearly named methods.
- Don't ignore return values: if `calculateTotal()` returns a value, use it.

## Control flow
- Prefer guard clauses over deep nesting.
- Guard clauses for invalid input THROW a meaningful exception; never silently `return` on null or invalid state in service code.
- Use `Objects.requireNonNull` / domain exceptions, not swallowed nulls.

## Duplication and simplicity
- Extract logic duplicated in 3+ places. Code that only looks similar but represents different business rules may stay separate.
- Prefer the JDK and existing libraries (`String.isBlank()`, `Duration`, streams) over custom helpers.
- No speculative features, config options or parameters "just in case".

## Values and types
- No magic numbers or strings: named constants, enums, or `Duration` / `BigDecimal` with clear names.
- Use small value-object records (`Email`, `Money`) where mixing up primitives is a real risk; validate in the compact constructor, including null.
- Prefer records and immutability; avoid setters on new types.

## Comments
- Comment WHY (business reason, workaround, ordering constraint), never WHAT the code does.
- If code needs a comment to be understood, rename or extract first.
- Delete dead and commented-out code; git keeps history.

## Review checklist
Names clear? One job per method? Nesting flattened? Errors thrown, not swallowed? Magic values named? Duplication real? Comments explain why?
