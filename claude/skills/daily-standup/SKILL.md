---
name: Daily Standup Briefing
description: Pulls open Jira tasks, Confluence pages where you are mentioned, and PR status into a concise standup-ready summary.
---

# Daily Standup Briefing

## Objective
Produce a standup-ready daily briefing covering Jira, Confluence, and pull requests.

## Steps

1. **Resolve Atlassian identity** using `atlassianUserInfo` — get the current user's accountId.

2. **Jira — open tasks:**
   - `assignee = currentUser() AND statusCategory != Done ORDER BY updated DESC`
   - `reporter = currentUser() AND statusCategory != Done ORDER BY updated DESC`
   - Capture: key, summary, status, priority, link.

3. **Confluence — recent activity:**
   - Pages modified by user: `contributor = currentUser() AND type = page ORDER BY lastmodified DESC` (limit 10)
   - Pages mentioning user: `type = page AND text ~ "<displayName>" ORDER BY lastmodified DESC` (limit 10)
   - Capture: title, space, last modified, link.

4. **Pull Requests:**
   - If a GitHub/GitLab connector is available: find open PRs authored by or review-requested for the user.
   - If unavailable: note this and suggest connecting one.

5. **Output the briefing:**

```markdown
# 📋 Daily Standup — {date}

## 🎯 Jira — Your Tasks
...

## 📝 Confluence — Recent Activity
...

## 🔀 Pull Requests
...

## ✅ Focus for Today
[2–3 sentence summary of highest-priority items]
```

## Constraints
- Keep each section concise — bullet points, not paragraphs.
- Flag anything overdue or blocked with ⚠️.
- If a section is empty, write "Nothing to action here."
