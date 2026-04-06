---
name: Java Unit Test Generator
description: Generates comprehensive JUnit 5 unit tests with Mockito for a given Java class or method. Covers happy paths, edge cases, and error scenarios.
---

# Java Unit Test Generator

## Objective
Generate a complete, ready-to-run JUnit 5 test class for the provided Java code.

## Steps

1. **Analyse the class/method** provided by the user:
   - Identify all public methods
   - Identify dependencies (constructor/field injection)
   - Note return types, exceptions thrown, and side effects

2. **Generate test cases** covering:
   - ✅ Happy path(s) — expected successful execution
   - 🔲 Edge cases — nulls, empty collections, boundary values, zero/max integers
   - ❌ Error scenarios — exceptions, validation failures, downstream failures

3. **Use this stack:**
   - JUnit 5 (`@Test`, `@ExtendWith`, `@ParameterizedTest` where appropriate)
   - Mockito (`@Mock`, `@InjectMocks`, `when(...).thenReturn(...)`, `verify(...)`)
   - AssertJ (`assertThat(...)`) preferred over JUnit assertions

4. **Output format:**
```java
@ExtendWith(MockitoExtension.class)
class YourClassTest {

    @Mock
    private Dependency dependency;

    @InjectMocks
    private YourClass yourClass;

    @Test
    void shouldDoSomethingWhenCondition() {
        // given
        // when
        // then
    }
}
```

## Constraints
- Follow Given/When/Then structure in every test method.
- Name tests descriptively: `shouldThrowExceptionWhenInputIsNull()`.
- Do not generate tests for private methods directly — test via public API.
- If the class has complex setup, generate a `@BeforeEach` method.
