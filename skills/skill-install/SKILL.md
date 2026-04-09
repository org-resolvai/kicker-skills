---
name: skill-install
description: Install, list, search, or remove agent skills. Supports searching the open skills ecosystem via `skills` CLI, installing from URLs (GitHub, playbooks.com, agentskills.io), or creating new skills from a user description.
---

# Skill Management

Manage skills in the workspace. Skills are reusable instructions (SKILL.md files) that teach the agent how to perform specific tasks.

## Commands

Respond to these user intents:

- **Search skills** — browse the open skills ecosystem
- **Install a skill** — from URL, search result, or description
- **List installed skills** — show all skills
- **Remove a skill** — delete a skill directory
- **Create a skill** — generate from user description

## Search & Browse Skills

The `skills` CLI (pre-installed) lets you search the open skills ecosystem.

### Browse a repository's skills

```bash
skills add <owner/repo> --list
```

Examples:

```bash
skills add anthropics/skills --list          # Official Anthropic skills
skills add vercel-labs/agent-skills --list   # Vercel skills
skills add <any-github-user/repo> --list     # Any repo with SKILL.md files
```

This shows all available skills in a repository with their names and descriptions.

### Install from search results

After browsing, the user may say "install the X skill from that repo". To install:

1. Convert the repo + skill name to a raw GitHub URL:
   `https://raw.githubusercontent.com/<owner>/<repo>/main/skills/<skill-name>/SKILL.md`
   or `https://raw.githubusercontent.com/<owner>/<repo>/main/<skill-name>/SKILL.md`
2. Use `fetch` to download the SKILL.md content
3. If the skill has additional files (referenced in SKILL.md), download those too
4. Extract `name` from YAML frontmatter
5. Write to `skills/<name>/SKILL.md`

Note: Do NOT use `skills add` to install — it writes to the wrong directory. Always use `fetch` + `write` to install into the workspace `skills/` directory.

## Install from URL

### GitHub URL

If the user gives a GitHub URL like `https://github.com/user/repo` or a specific file path:

1. Convert to raw content URL:
   - `https://github.com/user/repo/blob/main/SKILL.md` → `https://raw.githubusercontent.com/user/repo/main/SKILL.md`
   - If no file path, try `https://raw.githubusercontent.com/user/repo/main/SKILL.md`
2. Use `fetch` to download the content
3. Extract `name` from YAML frontmatter
4. Write to `skills/<name>/SKILL.md`

### playbooks.com URL

If the user gives a URL like `https://playbooks.com/skills/author/skills/skill-name`:

1. Convert to raw GitHub URL: `https://raw.githubusercontent.com/author/skills/main/skill-name/SKILL.md`
2. Use `fetch` to download
3. If the content is not a valid SKILL.md (no frontmatter), try `fetch` on the playbooks page itself and extract the skill content
4. Write to `skills/<name>/SKILL.md`

### agentskills.io URL

1. Use `fetch` on the URL to find the raw SKILL.md link
2. Download and install as above

### Direct raw URL

If the URL points directly to a `.md` file, just `fetch` and install.

## Install from Description

When the user describes what they want:

1. Generate a SKILL.md following this template:

```markdown
---
name: <lowercase-hyphenated-name>
description: <what the skill does and when to use it, max 1024 chars>
---

# <Skill Title>

<Clear step-by-step instructions for the agent>

## Steps

1. ...
2. ...

## Notes

- ...
```

2. The `name` must be: lowercase, letters/numbers/hyphens only, 1-64 chars, no leading/trailing/consecutive hyphens
3. Write to `skills/<name>/SKILL.md`

### Frontmatter Reference

| Field           | Required | Description                                             |
| --------------- | -------- | ------------------------------------------------------- |
| `name`          | Yes      | Must match directory name. Lowercase a-z, 0-9, hyphens. |
| `description`   | Yes      | What the skill does. Max 1024 chars.                    |
| `license`       | No       | License name.                                           |
| `compatibility` | No       | Environment requirements. Max 500 chars.                |
| `metadata`      | No       | Arbitrary key-value pairs (author, version, etc).       |
| `allowed-tools` | No       | Space-delimited pre-approved tool list.                 |

## List Skills

Use `find` to scan `skills/` for SKILL.md files, then `read` each to extract frontmatter:

```
skills/
├── email-summary/SKILL.md    — Summarize recent Gmail emails into a structured digest
├── gmail/SKILL.md            — Gmail email management: read, search, send, reply, draft, label, archive, and trash
├── google-calendar/SKILL.md  — Manage Google Calendar events
└── google-places/SKILL.md    — Search for businesses and places
```

Show name, description, and path for each.

## Remove a Skill

1. Confirm the skill name with the user
2. Delete with: `rm -rf skills/<name>/`
3. Confirm removal

## Handling Token / API Key Requirements

Some skills require API tokens or secrets. During installation, merge user-provided keys directly into the SKILL.md frontmatter so the skill file is fully self-contained.

### Detecting Key Requirements

After downloading a SKILL.md, scan its content for:

- Keywords like `ACCESS_TOKEN`, `API_KEY`, `SECRET`, `TOKEN`
- Template variables in `{{variable}}` or `<PLACEHOLDER>` format
- References to `integrations` tool for OAuth providers

### OAuth Integrations (Gmail, Google Calendar, Notion, etc.)

No user-provided key needed. The skill should use the `integrations` tool directly:

```
integrations({ provider: "gmail" })
```

Remind the user to connect the service in their app settings if not already authorized.

### Platform Built-in Tools

These tools have platform-managed API keys — no user configuration needed:
`web_search`, `places`, `fetch`, `image`, `tts`

### Third-party API Keys (Instagram, Slack, etc.)

Merge user-provided keys into the SKILL.md `metadata` field at install time:

1. **Prompt the user**: Tell them which keys are required and ask for input
2. **Write to frontmatter**: Store keys in the `metadata` object

```markdown
---
name: instagram
description: Post to Instagram
metadata:
  access_token: 'IGQVJWZAmQ2R3...'
---
```

3. **Reference in skill body**: The skill instructions should read the token from its own frontmatter `metadata.access_token`

This makes the skill file fully self-contained — no external config needed.

### Install Flow Example

```
User: Install https://github.com/someone/instagram-skill
Agent:
  1. fetch SKILL.md
  2. Detect ACCESS_TOKEN placeholder in content
  3. Ask user: "This skill requires an Instagram Access Token. Please provide it."
  4. User provides token
  5. Write token into metadata, replace placeholders
  6. Save to skills/instagram/SKILL.md
```

## After Install/Remove

Tell the user the skill will take effect on the **next message** — skills are reloaded at the start of each conversation.
