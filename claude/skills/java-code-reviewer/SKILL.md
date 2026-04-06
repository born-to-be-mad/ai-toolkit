---
name: Java Code Reviewer
description: Reviews Java code for quality, design patterns, SOLID principles, and common pitfalls. Provides actionable, prioritised feedback.
---

# Java Code Reviewer

## Objective
Review the provided Java code and produce a structured, actionable code review.

## Steps

1. **Read the code** provided by the user (paste, file, or PR diff).
2. **Evaluate across these dimensions:**
   - Correctness — logic errors, null safety, edge cases
   - Design — SOLID principles, separation of concerns, appropriate patterns
   - Readability — naming, method length, comments
   - Performance — unnecessary allocations, N+1 queries, blocking calls
   - Security — input validation, SQL injection, sensitive data exposure
   - Testability — can this be unit tested? Are dependencies injectable?
3. **Reference shared conventions** from `../../../../shared/conventions/java-style-guide.md` if available.
4. **Output the review** in this format:

```
## Code Review

### ✅ Strengths
[What is done well]

### 🔴 Critical Issues
[Must-fix before merge]

### 🟡 Suggestions
[Improvements worth considering]

### 🟢 Minor Nits
[Optional polish]

### Summary
[1–2 sentence verdict and merge recommendation]
```

## Constraints
- Be specific: reference line numbers or method names when possible.
- Be constructive: explain *why* something is an issue, not just *what*.
- Do not rewrite the entire code unless asked — focus on feedback.
