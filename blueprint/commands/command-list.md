---
name: command-list
description: "List custom slash commands in .claude/commands/"
---

List all custom slash commands defined in `.claude/commands/`. Do NOT use MCP tools.

**IMPORTANT: All output must be written in English.**

## Steps

1. List `.md` files in `.claude/commands/`
2. If none found, report "No commands defined yet"
3. For each file:
   - Extract name from filename (remove `.md`)
   - Read first line — if it's an HTML comment `<!-- ... -->`, extract as description
   - Check if content contains `$ARGUMENTS`
4. Display results

## Output Format

```
# Custom Commands

- `/project:<name>` — <description> (accepts arguments)

**Total:** N command(s)
```

$ARGUMENTS
