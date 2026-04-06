# GitHub Copilot — Java Conventions

> Copy this file content into `.github/copilot-instructions.md` in your Java project repo to apply these conventions globally in Copilot Chat.

## General

- Use Java 17+ features where appropriate (records, sealed classes, pattern matching, text blocks).
- Prefer immutability: `final` fields, `record` types for DTOs.
- Use `Optional` only as a return type — never as a field or parameter.

## Naming

- Classes: `PascalCase`, nouns (`OrderService`, `PaymentProcessor`).
- Methods: `camelCase`, verbs (`findById`, `calculateTotal`).
- Constants: `UPPER_SNAKE_CASE`.
- Test methods: `shouldDoSomethingWhenCondition()`.

## Spring Boot

- Annotate service classes with `@Service`, repositories with `@Repository`.
- Use constructor injection — never `@Autowired` on fields.
- Keep controllers thin: delegate all logic to services.
- Return `ResponseEntity<T>` from controllers.

## Testing

- JUnit 5 + Mockito + AssertJ.
- Given/When/Then structure in every test.
- One assertion concept per test method.

## Error Handling

- Define domain-specific exceptions (`OrderNotFoundException extends RuntimeException`).
- Use `@ControllerAdvice` for global exception handling.
- Never swallow exceptions with empty catch blocks.
