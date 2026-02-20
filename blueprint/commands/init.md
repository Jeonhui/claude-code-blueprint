---
name: init
description: "Initialize .claude/ directory structure for Claude Code Blueprint"
---

Initialize the `.claude/` directory structure for this project. Preserve all existing files and only create what's missing.

Perform the following steps using native file tools (Read, Write, Glob, Bash). Do NOT use MCP tools.

**IMPORTANT: All generated file content and output must be written in English.**

## Steps

1. Check which of these paths already exist using Bash (`test -d`, `test -f`) and Read:
   - `.claude/` directory
   - `.claude/CLAUDE.md`
   - `.claude/CLAUDE.local.md`
   - `.claude/.gitignore`
   - `.claude/rules/`
   - `.claude/skills/`
   - `.claude/agents/`
   - `.claude/commands/`
   - `.claude/mcp-internal/`
   - `CLAUDE.md` (root)

2. Create missing directories using Bash `mkdir -p`:
   - `.claude/rules`
   - `.claude/skills`
   - `.claude/agents`
   - `.claude/commands`
   - `.claude/mcp-internal`

3. Create missing files (skip if already exists):

   **`.claude/CLAUDE.md`** (project memory):
   ```
   # Project Guide

   > This file is automatically loaded by Claude Code as project-level memory.
   > Add project conventions, architecture decisions, and coding standards here.
   ```

   **`.claude/CLAUDE.local.md`** (local settings):
   ```
   # Local Settings

   > Personal settings (not committed to git).
   > Add your local preferences and overrides here.
   ```

   **`.claude/.gitignore`**:
   ```
   CLAUDE.local.md
   ```

4. Report what was created and what was skipped in a clean summary format:

```
# Blueprint Init Report

**Project:** <project root>

## Actions Taken
- <list of created files/directories>

## Skipped (Already Exists)
- <list of skipped items>
```

$ARGUMENTS
