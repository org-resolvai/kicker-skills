---
name: email-summary
description: Summarize recent Gmail emails into a structured digest. Read the gmail skill for API details.
---

# Email Summary Skill

Fetch and summarize the user's recent Gmail emails into a structured digest.

## How

1. Read the **gmail** skill (`skills/gmail/SKILL.md`) for Gmail API details (auth, endpoints, query syntax)
2. Use `format=metadata` to fetch emails — do not use `format=full` unless the user asks to read a specific email
3. Default to `newer_than:1d` if the user doesn't specify a time range

## Output

- Group by thread when possible
- Sort by date (most recent first)
- For each email: date, sender, subject, brief content summary (from snippet)
- Highlight actionable items (meetings, deadlines, requests)
- Prioritize actionable emails (requests, deadlines, meetings) over notifications
