---
name: status
description: "Report Blueprint status: memory layers, MCP servers, skills, agents, rules"
---

Report the current Blueprint status for this project. Do NOT use MCP tools — use native file tools only.

**IMPORTANT: All output must be written in English.**

## Steps

1. **Memory Layers** — Check existence of each:
   - `~/.claude/CLAUDE.md` (Global)
   - `CLAUDE.md` (Project root, legacy)
   - `.claude/CLAUDE.md` (Project memory)
   - `.claude/CLAUDE.local.md` (Local memory)

2. **MCP Servers** — Read `.mcp.json` if it exists. List each server name and command.

3. **Installed Skills** — List subdirectories in `.claude/skills/`. For each, check if `SKILL.md` exists.

4. **Installed Agents** — List `.md` files in `.claude/agents/`. For each, parse the YAML frontmatter to extract `model` and `tools`.

5. **Rules** — Count `.md` files in `.claude/rules/`.

## Output Format

```
# Blueprint Status Report

## Memory Layers
- [OK/MISSING] [LEVEL] Description

## MCP Servers
- Server name (command)

## Installed Skills
- Skill name (status)

## Installed Agents
- Agent name (model) (built-in/custom)

## Rules: N rule file(s) in .claude/rules/
```

$ARGUMENTS
