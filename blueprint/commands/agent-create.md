---
name: agent-create
description: "Create a custom agent definition file in .claude/agents/"
---

Create a custom agent definition. Parse the following from $ARGUMENTS:
- `name` (required): Agent name (lowercase, hyphens)
- `description` (required): Short description
- `model` (optional, default: sonnet): opus, sonnet, or haiku
- `tools` (optional): Comma-separated tool list (e.g., Read,Glob,Grep,Bash)
- `max_turns` (optional, default: 10): Maximum agentic turns. If omitted, use `maxTurns: 10` in frontmatter.

Arguments format: `<name> "<description>" [--model sonnet] [--tools Read,Glob,Grep] [--max-turns 10]`

Do NOT use MCP tools — use native file tools only.

**IMPORTANT: All generated file content and output must be written in English.**

## Steps

1. Parse the arguments
2. Check if `.claude/agents/<name>.md` already exists — if so, report and stop
3. Create `.claude/agents/` directory if needed
4. Ask the user for the system prompt content if not obvious from the description
5. Write the agent file with YAML frontmatter:

```markdown
---
name: <name>
model: <model>
tools: [<tools>]
maxTurns: <max_turns>
description: "<description>"
---

<system prompt content>
```

6. Report success with the file path

$ARGUMENTS
