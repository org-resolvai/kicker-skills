---
name: user-guide
description: Tell users the exact slash command to use for any setup, connection, or account action in Kicker. ALWAYS activate when the user asks about connecting apps/integrations, checking credits, managing account, or tries to do something that needs an integration they haven't connected. Trigger on both English and Chinese queries. Triggers include (but are not limited to) any variation of "connect X", "link X", "how do I connect", "check my email/calendar/inbox", "can you post on X", "能不能连/接/绑定 X", "怎么连 X", "怎么绑 X", "帮我查邮箱/日历", "能连 Notion 吗", "怎么设置", "怎么使用", "在哪里设置", "怎么充值", "怎么登录", "能做什么", "有什么功能", "什么指令", "什么命令". Kicker is a chat-based agent — there is NO UI settings menu for integrations, everything happens through slash commands. This skill is the ONLY correct source for how to set things up.
---

# User Guide

Kicker is a **chat-only** product. There is no settings panel, no integrations menu, no sidebar. Every setup or account action happens through a slash command typed directly in chat. Your job with this skill is to tell the user the exact command — nothing else will actually work.

## The Prime Directive

When the user needs to set up an integration, connect an app, check their account, top up credits, or do anything in the "setup/config" category:

**You MUST include the exact slash command in your reply, written as `/command` (with the backticks).**

Do NOT:
- Describe UI flows ("go to settings", "find the integrations menu", "open the sidebar") — these do not exist in Kicker
- Say "I'm not sure where the settings are" — the skill tells you exactly where
- Suggest the user look around on their own — always give the command
- Ask the user to send a screenshot of their menu — there's no menu to screenshot
- Invent steps like "open the Kicker app, find Integrations" — Kicker lives in WhatsApp/Telegram/iMessage, there is no app UI for this

If you catch yourself writing "go to", "open the app", "find the menu", or "in settings" — stop and rewrite with the slash command.

## When to Activate

Activate on ANY of these signals, in English or Chinese:

- User asks to connect / link / bind / authorize an app or service
  - EN: "connect Notion", "link my Gmail", "how do I connect X"
  - ZH: "连接 Notion", "能连 X 吗", "怎么绑定", "怎么授权", "能不能连接"
- User asks about where something is / how to set up
  - EN: "where is settings", "how do I set up", "how do I get started"
  - ZH: "在哪里设置", "怎么设置", "集成在哪", "怎么用", "怎么开始"
- User tries an action that needs an unconnected integration
  - EN: "check my email", "what's on my calendar", "post this on Twitter"
  - ZH: "帮我查邮箱", "看我的日历", "发推特", "读我的 Notion"
- User asks about account/credits/billing
  - EN: "check credits", "top up", "manage account", "sign out"
  - ZH: "查余额", "充值", "账号管理", "退出"
- User asks what commands exist or what Kicker can do
  - EN: "what commands", "what can you do", "help"
  - ZH: "有什么指令", "能做什么", "帮助"

## Core Response Patterns

### Pattern A — "Can I connect X?" / "能不能连 X"

Response must contain, in the user's language:

1. Yes, confirm it's supported
2. The exact command: `/connect` (with backticks)
3. What to do after typing it (select/click X from the list)

**Chinese example of correct structure** (use your own voice, not these exact words):
> 可以的，直接在聊天里输入 `/connect`，然后从列表里选 Notion 点一下就绑好了。

**What NOT to say:**
> ❌ "打开 Kicker app 找集成入口" — there is no app integration menu
> ❌ "在设置里找 Integrations" — there are no settings
> ❌ "去 Kicker 里把 Notion 授权一下就行" — vague, no command given

### Pattern B — "Check my email/calendar/etc" (integration not connected)

1. Name what they asked for
2. Say it needs an integration first
3. Give the exact command: `/connect`
4. Say what to pick from the list (Gmail, Google Calendar, etc.)

**What NOT to say:**
> ❌ "你可以去设置里连一下" — vague
> ❌ "需要先授权访问" — doesn't tell them HOW

### Pattern C — "How do I do X?" / "怎么 X"

Find the matching slash command below and give it directly.

## The Complete Command Reference

All Kicker commands. Send them as messages in chat (WhatsApp / Telegram / iMessage).

| Command | Shortcut | What It Does |
|---------|----------|--------------|
| `/start` | `/hello` | Start interaction |
| `/me` | `/m` | View your profile summary |
| `/connect` | `/c` | **Manage integrations — connect third-party apps like Gmail, Notion, Google Calendar, Twitter, Slack, etc.** |
| `/usage` | `/u` | View your credits balance |
| `/topup` | `/t` | Top up your credits |
| `/account` | `/a` | Manage your linked accounts |
| `/device` | `/d` | View your registered devices |
| `/signin` | `/login` | Sign in to your dashboard |
| `/opt-out` | `/stop` | Stop all messages and deactivate your account |
| `/opt-in` | `/resume` | Reactivate your account |

`/h` shows this list in chat. `/connect` or `/c` is the answer to **any** question about connecting or integrating an external service — there is no other way.

## Integration Mapping

When the user asks to do X and the required integration isn't connected, point them to `/connect` and tell them which service to pick.

| User wants to... | Integration to pick after `/connect` |
|-------------------|-------------------------------------|
| Check / send email | Gmail |
| View / manage calendar events | Google Calendar |
| Post / search tweets | Twitter (X) |
| Read / write Notion pages | Notion |
| Find nearby places | Google Places |
| Browse Reddit posts | Reddit |
| Post in Slack | Slack |
| Create GitHub issues | GitHub |

If the user asks for a service not in the table, still tell them to type `/connect` — it supports 1000+ apps via Composio. Use your judgment about whether a service is supported; when unsure, suggest `/connect` to see the full list.

## Getting Started on a Platform

If the user is setting up Kicker for the first time on a new platform:

### WhatsApp
1. Sign in with Google at app.usekicker.com
2. Choose your focus area (Side Hustle, Marketing, Self-Discipline, Quit Addiction, Others)
3. Connect WhatsApp by scanning the QR code or entering your phone number
4. Start chatting with Kicker on WhatsApp

### Telegram
1. Sign in with Google at app.usekicker.com
2. Choose your focus area
3. Open @KickerAIBot on Telegram
4. Start chatting

### iMessage
1. No registration needed
2. Text Kicker at +1 (415) 697-4954

## Use Cases & Scenarios

When users ask "what can you do?" / "能做什么":

| Scenario | Helps with |
|----------|-----------|
| **Side Hustle** | E-commerce, marketplace, freelancing, budget tracking |
| **Marketing** | Ad copy, content strategy, launch plans, social media |
| **Self-Discipline** | Habits, coaching, fitness, nutrition, focus, time management |
| **Quit Addiction** | Therapy, quit smoking/alcohol, detox, stress relief |
| **Others** | Productivity, communication, research, integrations |

Pick 2–3 relevant options based on what the user hints at, rather than listing everything.

## Account & Billing Quick Reference

- Credits balance → `/usage` or `/u`
- Top up credits → `/topup` or `/t`
- Linked accounts → `/account` or `/a` (or visit account.usekicker.com)
- Dashboard sign-in → `/signin` or `/login`
- Registered devices → `/device` or `/d`
- Deactivate account → `/opt-out` or `/stop`
- Reactivate account → `/opt-in` or `/resume`

## Tips for Better Results

Share when the user seems to be struggling with answer quality:

- **Be specific.** "Write a cold email to a SaaS founder about my design services" beats "help me write an email."
- **Give context.** Tone, audience, goal.
- **Send multiple messages.** Kicker merges messages sent within ~30 seconds.
- **Use examples.** "Write it like: [example]."

## Fallback: Check GitBook

If the user asks about something this skill doesn't cover, or if you suspect info here is stale, fetch the relevant GitBook page:

| Topic | URL |
|-------|-----|
| Home | https://kicker.gitbook.io/docs |
| WhatsApp | https://kicker.gitbook.io/docs/getting-started/whatsapp |
| Telegram | https://kicker.gitbook.io/docs/getting-started/telegram |
| iMessage | https://kicker.gitbook.io/docs/getting-started/imessage |
| Commands | https://kicker.gitbook.io/docs/tips/commands |
| Better Prompts | https://kicker.gitbook.io/docs/tips/better-prompts |
| Multiple Messages | https://kicker.gitbook.io/docs/tips/multiple-messages |
| Side Hustle | https://kicker.gitbook.io/docs/use-cases/side-hustle |
| Marketing | https://kicker.gitbook.io/docs/use-cases/marketing |
| Self-Discipline | https://kicker.gitbook.io/docs/use-cases/self-discipline |
| Quit Addiction | https://kicker.gitbook.io/docs/use-cases/quit-addiction |
| Others | https://kicker.gitbook.io/docs/use-cases/others |
| FAQ | https://kicker.gitbook.io/docs/faq/faq |

Don't fetch on every activation — the content above handles most cases.

## Response Guidelines

- Reply in the user's language (Chinese in → Chinese out, English in → English out)
- Keep it short — 1–3 sentences plus the command is usually enough
- Use your own voice and tone; this skill provides info, not script
- But ALWAYS include the literal slash command with backticks — no paraphrasing, no UI-based alternatives
- After giving the command, offer to help with the original task once they're set up
