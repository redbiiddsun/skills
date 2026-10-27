# skills

Claude Code skills by Sun.

| Skill | What it does |
|-------|--------------|
| [sun-code-standart](skills/sun-code-standart/SKILL.md) | No magic strings/numbers, no single-letter names, meaningful names |
| [clean-code](skills/clean-code/SKILL.md) | Clean code baseline for any language/framework: naming, functions, services, constants/enums, models, simplicity |

| Agent (plugin `review-loop`) | What it does |
|-------|--------------|
| [tech-lead](agents/tech-lead.md) | Reviews a just-made change (read-only), outputs a findings list or `No findings.` |
| [developer](agents/developer.md) | Fixes the tech lead's findings, reports per finding |

Review loop: after a code change, run `tech-lead`, pass its findings to `developer`; stop when the tech lead returns `No findings.`

## Install

**Option 1 — Claude Code plugin (recommended, easy to update)**

Run inside Claude Code:

```
/plugin marketplace add redbiiddsun/skills
/plugin install sun-code-standart@skills
/plugin install clean-code@skills
/plugin install review-loop@skills
```

Update later with `/plugin marketplace update skills`.

**Option 2 — copy the skill folder**

```bash
git clone https://github.com/redbiiddsun/skills.git
cp -R skills/skills/* ~/.claude/skills/
cp skills/agents/*.md ~/.claude/agents/
```

## Use

Ask Claude "apply the sun standard" or "clean up naming" — the skill triggers automatically.

## Add a new skill

1. Create `skills/<skill-name>/SKILL.md` (with `name` + `description` frontmatter).
2. Add a plugin entry for it in `.claude-plugin/marketplace.json`.

Agents go in `agents/<name>.md` and are listed under a plugin's `agents` array.
