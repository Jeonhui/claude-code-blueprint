---
name: agent-list
description: "List installed agents and available built-in agent templates"
---

List all installed agents and available built-in agent templates. Do NOT use MCP tools.

**IMPORTANT: All output must be written in English.**

## Steps

1. **Installed Agents** — List `.md` files in `.claude/agents/`. For each:
   - Parse YAML frontmatter (between `---` delimiters) to extract `model`, `tools`, `description`
   - Show first non-frontmatter, non-heading, non-empty line as preview

2. **Available Built-in Agents** — Show the registry with install status:

   | Name | Model | Description |
   |------|-------|-------------|
   | `code-reviewer` | sonnet | Code quality analysis with severity-rated feedback |
   | `debugger` | sonnet | Systematic debugging with root cause analysis |
   | `test-engineer` | sonnet | Test strategy design, test writing, and coverage analysis |
   | `architect` | opus | Architecture design advisory with trade-off analysis (read-only) |
   | `security-reviewer` | sonnet | Security vulnerability detection (OWASP Top 10) |

## Output Format

```
# Agent Library

## Installed Agents
- **name** (built-in/custom) — preview

## Available Agents (Built-in Registry)
- [installed/available] **name** (model): description
```

> Use `/blueprint:agent-add <name>` to install, or `/blueprint:agent-create` for custom agents.

$ARGUMENTS
