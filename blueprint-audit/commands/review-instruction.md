---
name: review-instruction
description: "Print role spec references and review commands for use in other agent contexts"
---

Print the auditor role spec file paths and review commands so the user can copy-paste them into another agent's context.

Do NOT use MCP tools — use native file tools only (Read, Glob).

**IMPORTANT: All output must be written in English.**

## Steps

1. Scan `.claude/agents/audit-*.md` for files with `type: auditor` in YAML frontmatter
2. For each auditor agent, extract: `name`, `focus`
3. Output the result in the format below

## Output

```
# Review Instruction

## Role Spec Files

Read the following agent role spec files to understand your review behavior:

| File | Focus |
|:---|:---|
| `.claude/agents/audit-logic.md` | <focus> |
| `.claude/agents/audit-style.md` | <focus> |
| ... (any custom auditor agents) | ... |

## Commands

​```bash
# Run review with all auditor agents
/blueprint-audit:review

# Run review with a specific auditor
/blueprint-audit:review -b <agent-name>

# Limit scan scope
/blueprint-audit:review -s "src/**/*.ts"

# Filter by minimum priority
/blueprint-audit:review -p high

# Preview only (no backlog files created)
/blueprint-audit:review --dry-run

# Combined example
/blueprint-audit:review -b logic -s "src/**/*.ts" -p high --dry-run
​```
```

$ARGUMENTS
