# Java Style Guide

Shared conventions referenced by both Claude skills and Copilot instructions. Update here — both agents pick it up automatically.

## Language Version
Target Java 17+. Use modern features: records, sealed classes, pattern matching, switch expressions, text blocks.

## Code Style

### Naming
- Classes → `PascalCase` nouns: `OrderService`, `PaymentProcessor`
- Methods → `camelCase` verbs: `findById`, `calculateDiscount`
- Variables → `camelCase`, descriptive: `customerOrders` not `list`
- Constants → `UPPER_SNAKE_CASE`: `MAX_RETRY_COUNT`
- Test methods → `shouldDoSomethingWhenCondition`

### Structure
- Max method length: ~20 lines. Extract if longer.
- Max class length: ~300 lines. Split by responsibility if longer.
- One public class per file.
- Package by feature, not by layer (e.g. `order/` not `service/`).

## Spring Boot
- Constructor injection only — no `@Autowired` on fields.
- Controllers are thin: delegate to services.
- Services contain business logic only — no HTTP concerns.
- Repositories handle persistence only.
- Return `ResponseEntity<T>` from REST controllers.

## Immutability
- Make fields `final` wherever possible.
- Use `record` for DTOs and value objects.
- Avoid `Optional` as field or parameter — use only as return type.

## Error Handling
- Custom exceptions per domain: `OrderNotFoundException`, `PaymentFailedException`.
- `@ControllerAdvice` for global HTTP error mapping.
- Never catch and swallow exceptions silently.
- Log at the boundary where the exception is handled, not where it's thrown.

## Testing
- Framework: JUnit 5 + Mockito + AssertJ
- Structure: Given / When / Then in every test
- One logical assertion per test
- Test names: `shouldReturnEmptyWhenNoOrdersExist`
- No `@Autowired` in unit tests — use `@ExtendWith(MockitoExtension.class)`
