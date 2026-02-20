---
name: agent-add
description: "Add a custom audit agent with a complete role specification"
---

Add a custom audit agent to the project's audit environment with a full role specification.

Parse from $ARGUMENTS:
- `--name <name>` or `-n <name>` (required): Agent name (lowercase letters, numbers, hyphens; must start with a letter)
- `--type <auditor|optimizer>` or `-t <type>` (required): Agent type
- `--from <path>` (optional): Import role spec from an existing markdown file instead of interactive creation
- `--minimal` (optional): Create with only required fields, skip interactive checklist gathering

Do NOT use MCP tools — use native file tools only (Read, Write, Glob, Bash).

**IMPORTANT: All generated file content and output must be written in English.**

## Auto-Init Check

Before executing the main logic, perform this initialization check:

1. Check if `.claude/audit/` directory exists using Glob.
2. If it does NOT exist, auto-initialize:
   - Create directories using `mkdir -p`:
     - `.claude/agents/`
     - `.claude/audit/backlog/`
     - `.claude/audit/archive/`
   - Create `.claude/audit/LEDGER.md`:
     ```
     # Audit Ledger

     | ID | Status | Fixed By | Target | Verified | Date |
     |:---|:---|:---|:---|:---|:---|
     ```
   - Create default agent files (audit-logic.md, audit-style.md, audit-fixer.md, audit-qa.md) as defined in the "Default Agent Definitions" section of `/blueprint-audit:review` (`blueprint-audit/commands/review.md`).
   - Report: "Audit environment initialized."
3. If `.claude/audit/` already exists, skip initialization silently.

## Validation

1. **Validate and normalize name:**
   - Must match `^[a-z][a-z0-9-]*$` (before prefix)
   - If invalid: "Invalid agent name `<name>`. Use lowercase letters, numbers, and hyphens. Must start with a letter."
   - **Auto-prefix:** If the name does not already start with `audit-`, prepend it. Example: `--name security` → file becomes `audit-security.md`, `name: audit-security` in frontmatter.
   - **Edge case:** If `--name audit` is provided (just the prefix word without a descriptor), treat it as invalid and report: "Please provide a descriptive name after `audit-`, e.g., `--name security`."
   - Report the final name: "Agent name: `audit-<name>`"

2. **Validate type:** Must be `auditor` or `optimizer`
   - If invalid: "Invalid type `<type>`. Must be `auditor` or `optimizer`."

3. **Check duplicates:** Check if `.claude/agents/audit-<name>.md` exists (use the prefixed name)
   - If exists: "Agent `audit-<name>` already exists at `.claude/agents/audit-<name>.md`. Use a different name or edit the existing file directly."

## Agent Creation

### Mode A: Import from file (`--from <path>`)
1. Read the specified file
2. Validate it has YAML frontmatter with at least `name` and `type`
3. Copy the content to `.claude/agents/<name>.md`
4. Report success

### Mode B: Interactive creation (default)
1. Ask the user for:
   - **Focus area**: One-line description of what this agent focuses on
   - **Review checklist** (for auditor) or **Optimization checklist** (for optimizer): List of specific items to check
   - **Scan patterns** (optional, for auditor): Glob patterns for files to scan
   - **Ignore patterns** (optional): Glob patterns for files to skip

2. Write the agent file to `.claude/agents/audit-<name>.md` with the complete role spec format:

   **For `auditor` type:**
   ```markdown
   ---
   name: audit-<name>
   type: auditor
   focus: "<focus area>"
   scan_patterns:
     - "<pattern1>"
     - "<pattern2>"
   ignore_patterns:
     - "<pattern1>"
   ---

   # <Name> Auditor

   You are a <name> auditor. <focus area description>.

   ## Review Checklist
   - <item 1>
   - <item 2>
   - ...

   ## Severity Criteria
   - **critical**: Directly causes failure, data loss, or security vulnerability
   - **high**: Likely to cause incorrect behavior under normal usage
   - **medium**: Could cause issues under edge cases or specific conditions
   - **low**: Code quality concern that may lead to issues during future changes

   ## Scan Instructions
   - Focus on source files matching the configured scan_patterns
   - For each issue, note the exact file path, line number, and a clear description
   - Rate priority using the Severity Criteria above
   - Provide a concise, actionable improvement suggestion for each issue
   ```

   **For `optimizer` type:**
   ```markdown
   ---
   name: audit-<name>
   type: optimizer
   focus: "<focus area>"
   ---

   # <Name> Optimizer

   You are a <name> optimizer. <focus area description>.

   ## Optimization Checklist
   - <item 1>
   - <item 2>
   - ...

   ## Fix Principles
   - Make minimal, focused changes — fix ONLY what the issue describes
   - Preserve existing code style and conventions
   - Do not introduce new dependencies unless absolutely necessary
   - Ensure changes do not break existing functionality

   ## Process
   1. Read the issue report to understand the problem and target file
   2. Read the target file and surrounding context
   3. Implement the change with minimal modifications
   4. Verify the change addresses the issue without side effects
   ```

### Mode C: Minimal creation (`--minimal`)
1. Ask the user only for the focus area (one line)
2. Generate a minimal agent file with default checklist items based on the focus area

## Report

```
# Agent Added

**Name:** audit-<name>
**Type:** <type>
**Focus:** <focus area>
**File:** .claude/agents/audit-<name>.md

> Use with review: `/blueprint-audit:review --by audit-<name>`
> Use with fix:    `/blueprint-audit:fix --by audit-<name> --next`
```

$ARGUMENTS
