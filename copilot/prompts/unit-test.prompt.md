---
mode: ask
description: Generate JUnit 5 unit tests for the selected Java class or method
---

Generate comprehensive JUnit 5 unit tests for the following Java code.

Requirements:
- Use Mockito for mocking dependencies
- Use AssertJ for assertions
- Follow Given/When/Then structure
- Cover: happy path, null inputs, empty collections, boundary values, and exception scenarios
- Name tests as `shouldDoSomethingWhenCondition()`
- Use `@ExtendWith(MockitoExtension.class)`

Code to test:
${selection}
