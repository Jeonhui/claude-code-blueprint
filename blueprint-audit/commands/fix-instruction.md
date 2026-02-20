---
name: fix-instruction
description: "Print role spec references and fix commands for use in other agent contexts"
---

Print the optimizer role spec file paths and fix commands so the user can copy-paste them into another agent's context.

Do NOT use MCP tools — use native file tools only (Read, Glob).

**IMPORTANT: All output must be written in English.**

## Steps

1. Scan `.claude/agents/audit-*.md` for files with `type: optimizer` in YAML frontmatter
2. For each optimizer agent, extract: `name`, `focus`
3. Output the result in the format below

## Output

```
# Fix Instruction

## Role Spec Files

Read the following agent role spec files to understand your fix behavior:

| File | Focus |
|:---|:---|
| `.claude/agents/audit-fixer.md` | <focus> |
| `.claude/agents/audit-qa.md` | <focus> |
| ... (any custom optimizer agents) | ... |

## Commands

​```bash
# Fix the highest-priority open issue
/blueprint-audit:fix --next

# Fix a specific issue by ID
/blueprint-audit:fix -i AUDIT-20260220-001

# Fix with a specific optimizer agent
/blueprint-audit:fix --next -b <agent-name>

# Fix + QA verification
/blueprint-audit:fix --next -v

# Preview only (no changes made)
/blueprint-audit:fix -i AUDIT-20260220-001 --dry-run

# Combined example
/blueprint-audit:fix --next -b fixer -v
​```

## Status & Dashboard

​```bash
# View full quality dashboard
/blueprint-audit:status

# One-line summary
/blueprint-audit:status --brief
​```
```

$ARGUMENTS
