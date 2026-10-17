# skills

Claude Code skills by Sun.

| Skill | What it does |
|-------|--------------|
| [sun-code-standart](skills/sun-code-standart/SKILL.md) | No magic strings/numbers, no single-letter names, meaningful names |

## Install

**Option 1 — Claude Code plugin (recommended, easy to update)**

Run inside Claude Code:

```
/plugin marketplace add redbiiddsun/skills
/plugin install sun-code-standart@skills
```

Update later with `/plugin marketplace update skills`.

**Option 2 — copy the skill folder**

```bash
git clone https://github.com/redbiiddsun/skills.git
cp -R skills/skills/sun-code-standart ~/.claude/skills/
```

## Use

Ask Claude "apply the sun standard" or "clean up naming" — the skill triggers automatically.

## Add a new skill

1. Create `skills/<skill-name>/SKILL.md` (with `name` + `description` frontmatter).
2. Add a plugin entry for it in `.claude-plugin/marketplace.json`.
