---
name: user-guide
description: Guide users on how to use Kicker — slash commands, integrations, platform setup, tips, and use cases. Activate when the user asks about available features, how to connect apps, how to get started, what commands exist, or when they attempt an action that requires an unconnected integration (e.g., asking to check email without Gmail connected, asking about calendar without Google Calendar connected). Also activate when the user seems lost, confused, or asks "what can you do".
---

# User Guide

Help users understand and use Kicker effectively by pointing them to the right commands, integrations, and features. Be concise — give users exactly what they need, not a documentation dump.

## When to Activate

- User tries to perform an action that requires an integration they haven't connected (e.g., "check my email" without Gmail, "what's on my calendar" without Google Calendar)
- User asks what Kicker can do, how it works, or what features are available
- User asks about slash commands or shortcuts
- User asks how to connect an app or service
- User asks about account management, credits, billing, or devices
- User asks how to get started on a new platform (WhatsApp, Telegram, iMessage)
- User asks what scenarios or use cases Kicker supports
- User seems confused, stuck, or new

## Missing Integration

This is the most common trigger. When the user asks to do something that requires an integration they haven't connected:

1. Check the user's current connected integrations
2. Identify which integration their request needs
3. Tell them how to connect it — keep it short and encouraging

**Your response should include:**

1. Acknowledge what the user wants to do
2. Explain that the required integration isn't connected yet
3. Tell them to type `/connect` and select the specific service
4. Briefly mention what you'll be able to do once it's connected

Keep it to 1–3 sentences. Use your own voice — don't follow a script. After the user connects the integration, proactively offer to retry their original request.

**Common mappings:**

| User wants to... | Needs integration |
|-------------------|-------------------|
| Check/send email | Gmail |
| View/manage calendar | Google Calendar |
| Post or search tweets | Twitter |
| Read/manage pages or databases | Notion |
| Find nearby places | Google Places |
| Browse subreddits or posts | Reddit |

## Asking to Connect a Specific Service

When the user asks "Can I connect X?" or "Do you support X?" or "How do I link my Calendar?":

1. Confirm that the integration is available (if you know it is)
2. Tell them to type `/connect`, then click on the specific service to bind it

**Your response should include:**

1. Confirm the service is available (if you know it is)
2. Tell them to type `/connect` and click on that specific service to bind it
3. Optionally mention what becomes possible after connecting

If you're unsure whether a specific service is supported, tell the user to type `/connect` to see the full list of available integrations.

## Slash Commands

All available commands. Users can type `/h` anytime to see this list in chat.

| Command | Shortcut | What It Does |
|---------|----------|--------------|
| `/start` | `/hello` | Start interaction |
| `/me` | `/m` | View your profile summary |
| `/connect` | `/c` | Manage integrations (connect third-party apps) |
| `/usage` | `/u` | View your credits balance |
| `/topup` | `/t` | Top up your credits |
| `/account` | `/a` | Manage your linked accounts |
| `/device` | `/d` | View your registered devices |
| `/signin` | `/login` | Sign in to your dashboard |
| `/opt-out` | `/stop` | Stop all messages and deactivate your account |
| `/opt-in` | `/resume` | Reactivate your account |

When a user asks about commands, share only the relevant ones — not the entire table. For example, if they ask "how do I check my credits?", just tell them about `/usage`.

## Getting Started

Platform-specific setup guidance for new users.

### WhatsApp
1. Sign in with Google at app.usekicker.com
2. Choose your focus area (Side Hustle, Marketing, Self-Discipline, Quit Addiction, or Others)
3. Connect WhatsApp by scanning the QR code or entering your phone number
4. Start chatting with Kicker on WhatsApp

### Telegram
1. Sign in with Google at app.usekicker.com
2. Choose your focus area
3. Connect Telegram — click the link to open @KickerAIBot
4. Start chatting — Telegram supports markdown formatting and has no 24-hour conversation window limit

### iMessage
1. No registration needed
2. Text Kicker at +1 (415) 697-4954
3. Start chatting directly

## Use Cases & Scenarios

When users ask what Kicker can help with, match their interest to a scenario:

| Scenario | What It Helps With | Example Skills |
|----------|-------------------|----------------|
| **Side Hustle** | Making money on the side | E-commerce, marketplace selling, freelancing, budget tracking |
| **Marketing** | Growing a brand or product | Ad creative, copywriting, content strategy, launch planning, social media |
| **Self-Discipline** | Building better habits | Habit tracking, coaching, fitness, nutrition, focus sessions, time management |
| **Quit Addiction** | Breaking unwanted habits | Therapy support, quit smoking/alcohol, social media detox, stress relief |
| **Others** | General productivity | Communication, academics, finances, research, app integrations |

If a user's interest doesn't clearly match one scenario, briefly describe the options and let them pick. Don't overwhelm — two or three relevant suggestions are better than listing everything.

## Tips for Better Results

Share these when a user seems to be struggling with response quality:

- **Be specific.** "Help me write a cold email to a SaaS founder about my design services" beats "help me write an email."
- **Give context.** Share relevant details — who it's for, what tone, what goal.
- **Send multiple messages.** Kicker merges messages sent within ~30 seconds, so add details as follow-ups instead of cramming everything into one message.
- **Use examples.** Show Kicker what you like: "Write it like this: [example]."

## Account & Billing

- **Check credits:** `/usage` or `/u`
- **Top up credits:** `/topup` or `/t`
- **Manage accounts:** `/account` or `/a` — or visit account.usekicker.com
- **Sign in to dashboard:** `/signin` or `/login`
- **View devices:** `/device` or `/d`

## Deactivation & Reactivation

- **Stop all messages and deactivate:** `/opt-out` or `/stop`
- **Reactivate your account:** `/opt-in` or `/resume`

## Fallback: Check GitBook for Latest Docs

If the user asks about something not covered above, or if you suspect the information here might be outdated, fetch the relevant GitBook page for the latest content:

| Topic | URL |
|-------|-----|
| Home / Overview | https://kicker.gitbook.io/docs |
| WhatsApp Setup | https://kicker.gitbook.io/docs/getting-started/whatsapp |
| Telegram Setup | https://kicker.gitbook.io/docs/getting-started/telegram |
| iMessage Setup | https://kicker.gitbook.io/docs/getting-started/imessage |
| Slash Commands | https://kicker.gitbook.io/docs/tips/commands |
| Better Prompts | https://kicker.gitbook.io/docs/tips/better-prompts |
| Multiple Messages | https://kicker.gitbook.io/docs/tips/multiple-messages |
| Side Hustle | https://kicker.gitbook.io/docs/use-cases/side-hustle |
| Marketing | https://kicker.gitbook.io/docs/use-cases/marketing |
| Self-Discipline | https://kicker.gitbook.io/docs/use-cases/self-discipline |
| Quit Addiction | https://kicker.gitbook.io/docs/use-cases/quit-addiction |
| Others | https://kicker.gitbook.io/docs/use-cases/others |
| FAQ | https://kicker.gitbook.io/docs/faq/faq |

Only fetch when needed — don't fetch on every activation. The content above should handle most cases.

## Response Guidelines

- Be concise — one or two sentences plus the relevant command is usually enough
- Use your own voice and tone as defined by your system prompt — this skill provides the information, not the wording
- Don't dump the entire guide; give them exactly what they need right now
- Don't invent commands or features that don't exist
- If unsure whether a specific integration is available, suggest `/connect` to see the full list
- After guiding the user, offer to help with their original task once they've completed the setup step
