# Kicker Skills

Markdown-first skill definitions for the [Kicker Agent](https://github.com/org-resolvai/kickerai) platform.

## Structure

```
skills/
  <skill-id>/
    SKILL.md          # Main skill prompt (required)
    skill.json        # Metadata (required)
    tools/            # Optional helper scripts
    assets/           # Optional assets
```

Each skill directory must contain at minimum:
- **`SKILL.md`** — the entry point prompt that the agent reads
- **`skill.json`** — machine-readable metadata

## Scenarios

Skills are grouped by onboarding scenario. See [`scenarios.json`](scenarios.json) for the full list.

| Scenario | Description |
|---|---|
| `base` | Pre-installed for every user |
| `side-hustle` | Freelancing, e-commerce, finance tracking |
| `marketing` | Ad creative, copywriting, content strategy |
| `self-discipline` | Habits, coaching, fitness, time management |
| `quit-addiction` | Sobriety support, stress relief, therapy |
| `others` | General-purpose skills |

## Contributing

1. Create a directory under `skills/` with a kebab-case name
2. Add `SKILL.md` with your prompt
3. Add `skill.json` following the schema below
4. Open a Pull Request

### `skill.json` schema

```json
{
  "id": "my-skill",
  "name": "My Skill",
  "description": "What this skill does in one sentence",
  "scenario": "others",
  "tags": ["tag1", "tag2"],
  "enabledByDefault": false,
  "source": "GitHub/ClawdHub/...",
  "installUrl": "https://...",
  "securityLevel": "LOW",
  "dependencyType": "none",
  "notes": "Any setup quirks"
}
```

## License

MIT
