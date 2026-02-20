---
name: hook-add
description: "Add an event hook to .claude/settings.json"
---

Add a hook to `.claude/settings.json`. Hooks run shell commands in response to Claude Code events.

Parse from $ARGUMENTS:
- `event` (required): PreToolUse, PostToolUse, Notification, Stop, SubagentStop, SessionStart, SessionStop. If the event is not one of these known events, warn the user but proceed anyway.
- `command` (required): Shell command to execute
- `--matcher <pattern>` (optional): Tool name or pattern to match (e.g., "Bash", "Edit"). Empty = all tools.
- `--timeout <ms>` (optional): Timeout in milliseconds

Example: `PreToolUse "echo hello" --matcher Bash --timeout 5000`

Do NOT use MCP tools — use native file tools only.

**IMPORTANT: All output must be written in English.**

## Steps

1. Parse arguments to extract event, command, matcher, timeout
2. Read `.claude/settings.json` (create if doesn't exist, start with `{}`)
3. Ensure `hooks` key exists in the JSON
4. Ensure `hooks.<event>` array exists
5. Find or create a matcher entry with the specified matcher value
6. Append the new hook `{ "type": "command", "command": "<cmd>", "timeout": <ms> }` to the matcher's hooks array
7. Write the updated JSON back to `.claude/settings.json` (pretty-printed with 2-space indent)
8. Report success:

```
# Hook Added

**Event:** <event>
**Matcher:** <matcher or "(all tools)">
**Command:** `<command>`
**Timeout:** <ms> (if specified)

> The hook will take effect on the next Claude Code session.
```

$ARGUMENTS
