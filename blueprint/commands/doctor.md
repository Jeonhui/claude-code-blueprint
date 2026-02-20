---
name: doctor
description: "Diagnose .claude/ structure integrity and optionally auto-fix issues"
---

Run diagnostics on the `.claude/` directory structure. Do NOT use MCP tools — use native file tools only.

**IMPORTANT: All generated file content and output must be written in English.**

If the argument contains "fix" or "auto-fix", automatically fix issues that can be auto-fixed.

## Diagnostic Checks

1. **`.claude/` directory** — Must exist. Auto-fixable: create it.
2. **`.claude/CLAUDE.md`** — Should exist (project memory).
3. **`.claude/.gitignore`** — Should exist and contain `CLAUDE.local.md`. Auto-fixable: create/update.
4. **Subdirectories** — Each should exist: `rules/`, `skills/`, `agents/`, `commands/`, `mcp-internal/`. Auto-fixable: create missing.
5. **Skills integrity** — Each subdirectory in `.claude/skills/` should have a `SKILL.md` file.
6. **Agents integrity** — Each `.md` file in `.claude/agents/` should have YAML frontmatter with `model:` field.
7. **`.mcp.json` validation** — If exists, must be valid JSON with `mcpServers` key. Check that internal server paths exist.
8. **`.claude/settings.json` validation** — If exists, must be valid JSON.

## Output Format

```
# Blueprint Doctor Report

**Project:** <path>
**Auto-fix:** enabled/disabled

## Diagnostics

- [OK/WARN/ERROR/FIXED] **item**: message

## Summary

| Status | Count |
| :--- | :---: |
| OK | N |
| Warning | N |
| Error | N |
| Fixed | N |
```

$ARGUMENTS
