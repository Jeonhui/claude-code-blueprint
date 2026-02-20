---
name: tool-link
description: "Register internal MCP servers into .mcp.json"
---

Register internal MCP servers from `.claude/mcp-internal/` into `.mcp.json`. Parse from $ARGUMENTS:
- `tool_name` (optional): Specific tool to link. If omitted, link all unlinked tools.

Do NOT use MCP tools — use native file tools only.

**IMPORTANT: All output must be written in English.**

## Steps

1. Check `.claude/mcp-internal/` exists — if not, report error
2. List subdirectories in `.claude/mcp-internal/`
3. If no tools found, report error
4. If `tool_name` specified, validate it exists in the directory
5. Read `.mcp.json` (create with `{ "mcpServers": {} }` if doesn't exist)
6. If `mcpServers` key does not exist, add it as an empty object
7. For each tool to link:
   - If already registered in `mcpServers`, skip
   - Otherwise add: `"<name>": { "command": "node", "args": [".claude/mcp-internal/<name>/index.js"] }`
8. Write updated `.mcp.json` (pretty-printed)
9. Report linked and skipped tools:

```
# Tool Link Report

## Linked (New)
- <tool> → .mcp.json

## Skipped (Already Registered)
- <tool>

**Total MCP servers in .mcp.json:** N

> Remember to run `npm install` in each tool directory before use.
```

$ARGUMENTS
