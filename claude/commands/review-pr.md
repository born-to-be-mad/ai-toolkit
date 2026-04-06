# /review-pr

Trigger a structured Java code review on a pull request diff or pasted code.

## Usage
```
/review-pr [paste code or PR link]
```

## What it does
Invokes the `java-code-reviewer` skill on the provided code. If a PR link is given and a GitHub connector is available, fetches the diff automatically.

Produces a prioritised review with Critical Issues, Suggestions, and a merge recommendation.
