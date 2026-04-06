---
name: Java Javadoc Writer
description: Writes clear, complete Javadoc comments for Java classes, interfaces, and methods. Follows standard Javadoc conventions.
---

# Java Javadoc Writer

## Objective
Generate Javadoc comments for the provided Java code.

## Steps

1. **Identify the target:** class, interface, enum, or specific methods.
2. **Write Javadoc** following these rules:
   - Start with a one-sentence summary (imperative mood: "Returns...", "Creates...", "Validates...")
   - Add a blank line, then a longer description if the behaviour is non-obvious
   - Document every `@param` with name and purpose
   - Document `@return` unless the method is `void`
   - Document all `@throws` / `@exception` with the condition that triggers them
   - Use `{@code ...}` for inline code references
   - Use `{@link ClassName#methodName}` for cross-references
3. **Do not** state the obvious (e.g. "Sets the name field to the given name value").
4. **Output** the original code with Javadoc inserted above each element.

## Example output format

```java
/**
 * Validates that the given order is eligible for discount application.
 *
 * <p>An order is eligible if it has at least one item, a non-null customer,
 * and a total value above the minimum threshold defined in application config.
 *
 * @param order the order to validate; must not be {@code null}
 * @return {@code true} if the order qualifies for a discount, {@code false} otherwise
 * @throws IllegalArgumentException if {@code order} is {@code null}
 */
public boolean isEligibleForDiscount(Order order) { ... }
```

## Constraints
- Keep the first sentence under 80 characters.
- Don't pad with filler phrases like "This method is used to...".
- For simple getters/setters, a one-liner is sufficient.
