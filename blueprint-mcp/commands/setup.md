---
name: setup
description: "Install dependencies and register generated MCP servers in .mcp.json"
---

Install dependencies and register generated MCP servers from `.claude/mcp-internal/` into `.mcp.json`.

Parse from $ARGUMENTS:
- `server_name` (optional): Specific server to set up. If omitted, sets up all unregistered servers.
- `--skip-install` (optional): Skip the `npm install` step.

Do NOT use MCP tools — use native file tools only (Read, Write, Glob, Bash).

**IMPORTANT: All output must be written in English.**

## Steps

1. Check `.claude/mcp-internal/` exists — if not, report error:
   > No `.claude/mcp-internal/` directory found. Run `/blueprint-mcp:generate` first.

2. List subdirectories in `.claude/mcp-internal/`

3. If no servers found, report error:
   > No generated MCP servers found in `.claude/mcp-internal/`.

4. If `server_name` specified, validate it exists. If not found, list available servers.

5. For each server to set up:

   **a. Install dependencies** (unless `--skip-install`):
   - Check if `package.json` exists in the server directory
   - If yes, run `npm install --production` in that directory
   - Report success or failure

   **b. Register in `.mcp.json`**:
   - Read `.mcp.json` from project root (create with `{}` if missing)
   - Ensure `mcpServers` key exists
   - If server already registered, skip
   - Otherwise add:
     ```json
     "<server-name>": {
       "command": "node",
       "args": [".claude/mcp-internal/<server-name>/index.js"]
     }
     ```
   - Write updated `.mcp.json` (pretty-printed)

6. Report results:

```
# MCP Server Setup Report

### <server-name>
- npm install: done | skipped | FAILED - <error>
- .mcp.json: registered | already registered (skipped)

**Total MCP servers in .mcp.json:** N

> Restart Claude Code to activate the new MCP servers.
```

$ARGUMENTS
