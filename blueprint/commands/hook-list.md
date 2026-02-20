---
name: hook-list
description: "List all configured hooks in .claude/settings.json"
---

List all configured hooks grouped by event type. Do NOT use MCP tools.

**IMPORTANT: All output must be written in English.**

## Steps

1. Read `.claude/settings.json`
2. If file doesn't exist or has no `hooks` key, report "No hooks configured". If the file exists but contains invalid JSON, report: "settings.json contains invalid JSON. Please fix it manually or re-create it." and stop.
3. For each event in `hooks`:
   - For each matcher entry:
     - Show the matcher pattern (or "(all tools)" if empty)
     - List each hook's command and timeout
4. Show total count

## Output Format

```
# Configured Hooks

## <EventName>

**Matcher:** `<pattern>`
- `<command>` (timeout: <ms>)

**Total:** N hook(s) across M event(s)
```

$ARGUMENTS
