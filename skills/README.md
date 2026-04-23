---
title: Lynko Skills
audience: users
last_validated: 2026-04-23
---

# Lynko Skills

Agent skills for common Lynko workflows. These follow the [Agent Skills](https://agentskills.io) open standard, the same format used by Claude Code, OpenAI Codex, GitHub Copilot, and Cursor.

## Available Skills

| Skill | Description |
|-------|-------------|
| [explore](explore/) | Navigate collections, read content, search code |
| [develop](develop/) | Full dev workflow: read, edit, test, commit |
| [review](review/) | Review changes, read diffs, cross-reference |
| [run](run/) | Execute commands on a target machine (builds, scripts, smoke tests) via a runner node |

## Installation

### Claude Code

```bash
# Option A: Add the repo as a skill directory
git clone https://github.com/lynko-ai/lynko.git ~/lynko-skills
# Skills in ~/lynko-skills/skills/ are auto-discovered with --add-dir

# Option B: Copy individual skills
cp -r lynko/skills/explore ~/.claude/skills/lynko-explore
```

### OpenAI Codex CLI

```bash
cp -r lynko/skills/explore ~/.codex/skills/lynko-explore
```

### GitHub Copilot (VS Code)

Copy skill folders into your project's `.github/skills/` directory, or configure a custom skill location in VS Code settings.

### Claude.ai (Web / Mobile)

Upload the skill folder as a zip file via Settings → Features → Skills.

### Through Lynko Itself

If you've added this repo as a Lynko collection, your agent can read any skill directly:

```
lynko[skills/explore/SKILL.md].read()
```

The skill content is immediately available in context — no installation needed.

## Creating Your Own Skills

Lynko skills are standard SKILL.md files. Create a folder with a SKILL.md containing YAML frontmatter (`name` and `description`) and markdown instructions. See [agentskills.io/specification](https://agentskills.io/specification) for the full spec.