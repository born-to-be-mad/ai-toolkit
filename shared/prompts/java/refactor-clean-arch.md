# Prompt: Refactor to Clean Architecture

**Agent-agnostic.** Use with Claude, Copilot, or any LLM.

---

Refactor the following Java code towards Clean Architecture principles:

- Separate **domain logic** from infrastructure concerns (DB, HTTP, messaging)
- Introduce **use cases / application services** for each business operation
- Define **domain entities** and **value objects** — make them infrastructure-free
- Use **ports and adapters** (interfaces in domain, implementations in infra)
- Ensure the **dependency rule**: outer layers depend on inner layers, never the reverse

Steps:
1. Identify the core domain concept in the code.
2. Extract a pure domain model (no Spring annotations, no JPA annotations).
3. Define a use case interface and implementation.
4. Move persistence/HTTP concerns to an adapter layer.
5. Show the final package structure.

Preserve all existing behaviour. Do not change business logic — only restructure.
