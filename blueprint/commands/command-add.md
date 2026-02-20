---
name: command-add
description: "Create a custom slash command in .claude/commands/"
---

Create a custom slash command. Parse from $ARGUMENTS:
- `name` (required): Command name (lowercase, hyphens allowed)
- `prompt` (required): The prompt template content
- `--description <text>` (optional): Short description

Example: `my-command "Analyze the codebase for performance issues" --description "Performance analysis"`

Do NOT use MCP tools — use native file tools only.

**IMPORTANT: All generated file content and output must be written in English.**

## Steps

1. Parse arguments
2. Validate name: lowercase letters, numbers, hyphens only
3. Check if `.claude/commands/<name>.md` already exists — if so, report and stop
4. Create `.claude/commands/` directory if needed
5. Write the command file:
   - If description provided: `<!-- <description> -->\n\n<prompt>\n`
   - Otherwise: `<prompt>\n`
6. Report success:

```
# Command Created: <name>

**Usage:** `/project:<name>`
**Description:** <description>
**Location:** .claude/commands/<name>.md

> The command will be available in the next Claude Code session as `/project:<name>`.
> Use `$ARGUMENTS` in the prompt to accept user input.
```

$ARGUMENTS
