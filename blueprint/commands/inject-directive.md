---
name: inject-directive
description: "Inject Inherited Blueprint directive into root CLAUDE.md"
---

Inject (or update) a CCB directive into the root `CLAUDE.md` that instructs AI agents to follow the `.claude/` configuration. Do NOT use MCP tools.

**IMPORTANT: All generated file content and output must be written in English.**

Options in $ARGUMENTS:
- `--dry-run` to preview without writing

## Steps

1. **Scan installed components**:
   - Check `.claude/CLAUDE.md` exists
   - Check `.claude/CLAUDE.local.md` exists
   - List `.md` files in `.claude/rules/`
   - List subdirectories in `.claude/skills/`
   - List `.md` files in `.claude/agents/`
   - List subdirectories in `.claude/mcp-internal/`
   - Check `.mcp.json` exists

2. **Build directive content**:
   - Section: Project Guide (links to .claude/CLAUDE.md, CLAUDE.local.md)
   - Section: Rules (list all rule files)
   - Section: Domain Skills (list all skill directories)
   - Section: Agent Definitions (list all agent files)
   - Section: MCP Servers (list servers from .mcp.json)
   - Footer with generation note

3. **Write to root CLAUDE.md using block preservation**:
   - Wrap ALL generated content between markers:
     ```
     <!-- CCB:DIRECTIVE:BEGIN -->
     (generated content here)
     <!-- CCB:DIRECTIVE:END -->
     ```
   - **If file exists AND contains `<!-- CCB:DIRECTIVE:BEGIN -->`**: replace ONLY the content between `BEGIN` and `END` markers. Preserve everything outside the markers untouched.
   - **If file exists but does NOT contain the markers**: append the marker block to the end of the file. Do NOT modify existing content.
   - **If file does not exist**: create the file with the marker block.
   - If `--dry-run`, show preview only.

4. Report what was done (created new / updated existing block / appended to existing).

$ARGUMENTS
