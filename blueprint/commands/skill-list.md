---
name: skill-list
description: "List installed skills and available built-in skills"
---

List all installed skills and available built-in skills. Do NOT use MCP tools.

**IMPORTANT: All output must be written in English.**

## Steps

1. **Installed Skills** — List subdirectories in `.claude/skills/`. For each:
   - Check if `SKILL.md` exists
   - Check if `resources/` directory exists
   - Show first non-heading, non-empty line of `SKILL.md` as preview

2. **Available Built-in Skills** — Show the registry of built-in skills with install status:

   | Name | Description |
   |------|-------------|
   | `rest-api-guide` | REST API design conventions and best practices |
   | `test-convention` | Testing conventions and best practices |
   | `typescript-strict` | Strict TypeScript coding standards |
   | `git-workflow` | Git workflow and commit conventions |

## Output Format

```
# Skill Library

## Installed Skills
- skill-name [OK/MISSING SKILL.md] [has resources]

## Available Skills (Built-in Registry)
- [installed/available] **name**: description
```

> Use `/blueprint:skill-add <name>` to install a skill.

$ARGUMENTS
