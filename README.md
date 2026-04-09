# Kicker Skills

Skill library for [Kicker Agent](https://github.com/org-resolvai/kickerai). Each skill is a directory with a `SKILL.md` prompt file that the agent reads at runtime.

## How it works

```
This repo                          Sandbox container
────────────                       ─────────────────
skills/gmail/SKILL.md    ──build──> /app/kicker-skills/skills/gmail/SKILL.md
skills/ecommerce/SKILL.md          /app/kicker-skills/skills/ecommerce/SKILL.md
scenarios.json                     /app/kicker-skills/scenarios.json
                                          │
                                   cold start: skill-loader reads SCENARIOS env
                                   e.g. SCENARIOS=base,hustle
                                          │
                                          ▼
                                   workspace/skills/gmail/SKILL.md       (base)
                                   workspace/skills/ecommerce/SKILL.md   (hustle)
                                          │
                                          ▼
                                   agent scans skills → matches task → reads SKILL.md → executes
```

1. **Docker build** clones this repo into the sandbox image at `/app/kicker-skills/`
2. **Cold start** — `skill-loader.ts` reads the `SCENARIOS` env var (e.g. `base,hustle`), looks up `scenarios.json` to find which skill IDs belong to those scenarios, and copies the matching directories into the user's workspace
3. **Runtime** — `skill-registry.ts` scans `workspace/skills/`, parses each `SKILL.md` frontmatter, and injects the skill list into the agent's system prompt. The agent reads the full `SKILL.md` on demand when a task matches

## Directory structure

```
kicker-skills/
├── scenarios.json              # Scenario → skill ID mapping (single source of truth)
├── README.md
└── skills/
    ├── gmail/                  # Single-file skill
    │   └── SKILL.md
    ├── ecommerce/              # Multi-file skill
    │   ├── SKILL.md            # Entry point (required)
    │   ├── growth.md           # Referenced by SKILL.md
    │   ├── operations.md
    │   ├── platforms.md
    │   └── code-traps.md
    └── ...
```

**Each skill directory must have a `SKILL.md` at the root.** This is the only hard requirement. Additional `.md` files, scripts, or assets are optional — the agent can read them via relative paths referenced in `SKILL.md`.

No `skill.json` or other metadata files. Grouping is managed centrally in `scenarios.json`.

## SKILL.md format

Uses YAML frontmatter (parsed by [gray-matter](https://github.com/jonschlinkert/gray-matter)):

```markdown
---
name: my-skill
description: One-line summary of what the skill does.
---

# My Skill

Instructions for the agent...
```

**Required frontmatter fields:**
- `name` — lowercase, kebab-case, must match the directory name (e.g. `gmail`)
- `description` — one sentence, shown in the agent's skill list

**Optional frontmatter fields:**
- `allowed-tools` — space-separated tool names the skill is allowed to use
- `disable-model-invocation` — `true` to hide from the agent's auto-matching (user must invoke explicitly)
- `compatibility` — version/platform requirements
- `license` — license identifier

**Body** — free-form markdown. This is what the agent reads when executing the skill. Write it as instructions: what to do, what tools to call, what format to output, edge cases, etc. For large skills, split into multiple `.md` files and reference them: "See `growth.md` for growth strategies."

## scenarios.json

Maps scenario IDs to arrays of skill directory names:

```json
{
  "base": ["gmail", "translate", "..."],
  "hustle": ["ecommerce", "marketplace", "..."],
  "marketing": ["ad-creative", "copywriting", "..."]
}
```

| Scenario | When installed | Skills |
|---|---|---|
| `base` | Always (every user) | Core productivity — email, calendar, translate, etc. |
| `hustle` | User selected "Side Hustle" during onboarding | E-commerce, freelance, finance tracking |
| `marketing` | User selected "Marketing" | Ad creative, copywriting, content strategy |
| `discipline` | User selected "Self-Discipline" | Habits, coaching, fitness, time management |
| `addiction` | User selected "Quit Addiction" | Sobriety support, therapy, stress relief |
| `other` | User selected "Others" or entered via IM directly | General-purpose (e.g. Composio connect) |

A skill can appear in multiple scenarios. `base` is always included regardless of the user's selection.

## Adding a new skill

1. Create a directory under `skills/` with a kebab-case name:
   ```
   skills/my-new-skill/SKILL.md
   ```

2. Write `SKILL.md` with frontmatter + instructions:
   ```markdown
   ---
   name: my-new-skill
   description: What this skill does in one line.
   ---

   # My New Skill

   When the user asks to [do X], follow these steps:
   1. ...
   2. ...
   ```

3. Add the skill ID to the appropriate scenario in `scenarios.json`:
   ```json
   {
     "marketing": ["ad-creative", "...", "my-new-skill"]
   }
   ```

4. Open a Pull Request. After merge, the next sandbox image build will include the new skill.

## Modifying an existing skill

Edit the `SKILL.md` (or sub-files) directly and open a PR. No version bumping, no metadata updates — just change the content.

After merge + sandbox image rebuild, new containers will get the updated skill. Existing running containers keep the old version until restarted. Users who have customized the skill locally will not be overwritten (the skill-loader detects customizations via content hash).

## Moving a skill between scenarios

Edit `scenarios.json` only. No changes to the skill directory needed.

## Removing a skill

1. Remove the skill ID from all scenario arrays in `scenarios.json`
2. Optionally delete the `skills/<id>/` directory (or keep it as an archive)
3. After deployment, the skill-loader will remove it from new containers' workspaces on next cold start

## Local development

To test a skill without deploying:

```bash
# Copy your skill into a running sandbox's workspace
docker cp skills/my-new-skill/ <container>:/root/.kicker/workspace/skills/my-new-skill/

# The agent will pick it up on next message (skill-registry reloads on each run)
```

## Constraints

- `name` in frontmatter must be lowercase kebab-case, 1–64 characters, match the directory name
- `description` must be present and under 1024 characters
- `SKILL.md` must be under 100 KB (larger files are skipped by skill-registry)
- No executable code is auto-run — the agent reads markdown and decides what to do. Scripts in the skill directory are only executed if the agent explicitly calls them via bash tool

## License

MIT
