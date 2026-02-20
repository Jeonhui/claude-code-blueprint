---
name: export
description: "Export .claude/ directory index for other AI agents"
---

Export the `.claude/` directory index as guide files for other AI agents. Do NOT use MCP tools.

**IMPORTANT: All generated file content and output must be written in English.**

$ARGUMENTS specifies the target (default: `all`):
- `codex` → `AGENTS.md` (OpenAI Codex)
- `gemini` → `GEMINI.md` (Google Gemini)
- `copilot` → `.github/copilot-instructions.md` (GitHub Copilot)
- `all` → generate all three

If no target is specified, default to `all`.

Add `--dry-run` to preview without writing.

## Steps

1. **Build directory index** by scanning:
   - `.claude/CLAUDE.md`, `.claude/CLAUDE.local.md`
   - `.claude/rules/*.md`
   - `.claude/skills/*/` (check for SKILL.md, resources/)
   - `.claude/agents/*.md`
   - `.claude/mcp-internal/*/`
   - `.mcp.json`

2. **Format the index** as a Markdown guide:
   - Header explaining CCB and `.claude/` directory
   - Section for each component type with file paths
   - Directory tree overview

3. **Write target file(s) using block preservation**:
   - Wrap ALL generated content between markers:
     ```
     <!-- CCB:EXPORT:BEGIN -->
     (generated content here)
     <!-- CCB:EXPORT:END -->
     ```
   - **If file exists AND contains `<!-- CCB:EXPORT:BEGIN -->`**: replace ONLY the content between `BEGIN` and `END` markers. Preserve everything outside the markers untouched.
   - **If file exists but does NOT contain the markers**: append the marker block to the end of the file. Do NOT modify existing content.
   - **If file does not exist**: create the file with the marker block.
   - Create parent directories if needed (e.g., `.github/`)
   - If `--dry-run`, show preview only.

4. Report results for each file (created new / updated existing block / appended to existing).

$ARGUMENTS
