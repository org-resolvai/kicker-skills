---
name: user-guide
description: Tell users the exact slash command to use for any setup or integration action in Kicker. Activate whenever the user expresses intent to connect/link/authorize a third-party service (Notion, Gmail, Calendar, Twitter, Slack, any other app), asks how to set something up, asks where settings are, tries to use a feature that needs an unconnected integration (e.g. checking email, reading calendar, posting on social), asks about their account, credits, billing, or devices, or asks what Kicker can do. Recognize the intent regardless of language — users may write in English, Chinese, Spanish, Portuguese, Japanese, or anything else. Kicker is chat-only (WhatsApp / Telegram / iMessage) with no UI menu, so the answer to "how do I set this up?" is always a slash command — this skill is the authoritative source for which one.
---

# User Guide

Kicker runs inside chat apps (WhatsApp, Telegram, iMessage). It has **no UI**: no settings panel, no integrations menu, no sidebar, no app screen to click through. Every setup, integration, and account action happens by typing a slash command directly in chat.

Your job with this skill is simple: when the user needs to do a setup/config thing, tell them the exact command.

## The Prime Directive

When the user's intent is "I want to set up / connect / configure X":

**Your reply MUST contain the literal slash command, written as `/command` (with backticks).**

Do NOT:
- Describe UI flows ("go to settings", "find the integrations menu", "open the sidebar", "open the app", "check the left panel") — none of that exists
- Say you're "not sure where settings are" — you know the answer: it's a command
- Ask the user to send a screenshot of their menu
- Invent multi-step app navigation

If you notice yourself writing phrases like "go to", "open the app", "find the menu", or "in settings" — stop and rewrite using the slash command.

**Reply in whatever language the user wrote in.** If they wrote Chinese, reply in Chinese. Spanish in → Spanish out. Japanese in → Japanese out. The skill instructions here are in English, but the user-facing response should match the user.

## Core Scenarios

### User asks to connect / link / integrate a service

The answer is always: type `/connect`, then pick the service from the list that appears.

Your response should contain, in the user's language:
1. Confirmation that the service is supported (or that they should check the list)
2. The literal `/connect` command (with backticks)
3. Brief mention of what to do in the list (pick the service)

### User tries to use a feature that needs an unconnected integration

(e.g. "check my email" but Gmail isn't connected; "what's on my calendar" but Google Calendar isn't connected)

Response structure:
1. Acknowledge what they want to do
2. Note the required integration isn't connected yet
3. Tell them to send `/connect` and pick the service
4. Offer to help once they've connected

### User asks "how do I set up X" or "where is the setting for X"

There is no settings panel. Find the matching slash command below and give it directly.

### User asks "what can you do" / "what are you"

Briefly describe 2–3 relevant scenarios matching their hint. Don't dump the full list.

## All Commands

Send these as regular chat messages.

| Command | Shortcut | What It Does |
|---------|----------|--------------|
| `/start` | `/hello` | Start interaction |
| `/me` | `/m` | View your profile summary |
| `/connect` | `/c` | **Manage integrations — connect apps (Gmail, Notion, Calendar, Twitter, Slack, and 1000+ more)** |
| `/usage` | `/u` | View your credits balance |
| `/topup` | `/t` | Top up your credits |
| `/account` | `/a` | Manage your linked accounts |
| `/device` | `/d` | View your registered devices |
| `/signin` | `/login` | Sign in to your dashboard |
| `/opt-out` | `/stop` | Stop all messages and deactivate account |
| `/opt-in` | `/resume` | Reactivate account |

`/h` shows this list in chat. **`/connect` is the universal answer for connecting any third-party service** — there is no other path.

## Integration Mapping

When a user's feature request needs a specific integration:

| User wants to... | Integration to pick after `/connect` |
|-------------------|-------------------------------------|
| Read / send email | Gmail |
| View / manage calendar events | Google Calendar |
| Post / search tweets | Twitter (X) |
| Read / write Notion pages or databases | Notion |
| Find nearby places or businesses | Google Places |
| Browse Reddit posts or subreddits | Reddit |
| Post in Slack | Slack |
| Create GitHub issues or PRs | GitHub |

For any service not listed, still direct them to `/connect` — Kicker supports 1000+ integrations via Composio. If you're unsure whether a specific service is supported, tell the user to `/connect` to browse the full list.

## Platform Getting Started (first-time setup)

### WhatsApp
1. Sign in with Google at app.usekicker.com
2. Choose a focus area (Side Hustle, Marketing, Self-Discipline, Quit Addiction, Others)
3. Scan the QR code or enter your phone number
4. Start chatting on WhatsApp

### Telegram
1. Sign in at app.usekicker.com
2. Choose a focus area
3. Open @KickerAIBot on Telegram
4. Start chatting

### iMessage
1. Text +1 (415) 697-4954 — no registration needed

## Use Cases

| Scenario | Helps with |
|----------|-----------|
| Side Hustle | E-commerce, marketplace, freelancing, budget tracking |
| Marketing | Ad copy, content strategy, launch plans, social media |
| Self-Discipline | Habits, coaching, fitness, nutrition, focus, time management |
| Quit Addiction | Therapy, quit smoking/alcohol, detox, stress relief |
| Others | General productivity, communication, research, integrations |

Match 2–3 relevant scenarios to the user's hint rather than listing all.

## Account & Billing Quick Reference

- Credits balance → `/usage`
- Top up → `/topup`
- Linked accounts → `/account` (or account.usekicker.com)
- Dashboard sign-in → `/signin`
- Devices → `/device`
- Deactivate → `/opt-out`
- Reactivate → `/opt-in`

## Tips for Better Results

Share when the user's request is vague:
- Be specific about goal, audience, tone
- Provide context and examples
- Kicker merges messages sent within ~30 seconds — you can add follow-ups naturally

## Fallback: GitBook

For topics not covered above, or if info here feels outdated, fetch the relevant page:

| Topic | URL |
|-------|-----|
| Home | https://kicker.gitbook.io/docs |
| WhatsApp setup | https://kicker.gitbook.io/docs/getting-started/whatsapp |
| Telegram setup | https://kicker.gitbook.io/docs/getting-started/telegram |
| iMessage setup | https://kicker.gitbook.io/docs/getting-started/imessage |
| Slash commands | https://kicker.gitbook.io/docs/tips/commands |
| Better prompts | https://kicker.gitbook.io/docs/tips/better-prompts |
| Multiple messages | https://kicker.gitbook.io/docs/tips/multiple-messages |
| Side Hustle | https://kicker.gitbook.io/docs/use-cases/side-hustle |
| Marketing | https://kicker.gitbook.io/docs/use-cases/marketing |
| Self-Discipline | https://kicker.gitbook.io/docs/use-cases/self-discipline |
| Quit Addiction | https://kicker.gitbook.io/docs/use-cases/quit-addiction |
| Others | https://kicker.gitbook.io/docs/use-cases/others |
| FAQ | https://kicker.gitbook.io/docs/faq/faq |

Don't fetch on every activation — the content above handles the common cases.

## Response Guidelines

- Reply in the user's language — match what they wrote in, regardless of what language this skill file is in
- Keep it short: 1–3 sentences plus the command
- Use your own voice; this skill provides information, not wording
- Always include the literal slash command with backticks — never paraphrase, never describe a UI alternative
- After giving the command, offer to help with the user's original task once they're set up
