# Prompt: Java Code Review

**Agent-agnostic.** Use this prompt with Claude, Copilot, or any LLM.

---

Review the following Java code. Evaluate it across these dimensions:

1. **Correctness** — logic errors, null safety, unhandled edge cases
2. **Design** — SOLID principles, appropriate use of patterns, separation of concerns
3. **Readability** — naming clarity, method length, comments
4. **Performance** — unnecessary allocations, blocking calls, inefficient loops
5. **Security** — input validation, sensitive data exposure, injection risks
6. **Testability** — are dependencies injectable? can this be unit tested?

Structure your response as:
- ✅ Strengths
- 🔴 Critical Issues (must fix before merge)
- 🟡 Suggestions (worth improving)
- 🟢 Minor Nits (optional polish)
- Verdict: merge / needs work / reject

Be specific — reference method names and line numbers. Be constructive — explain *why*, not just *what*.
